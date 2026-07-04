# AWS RDS (Relational Database Service)

#technology #aws

**RDS** runs managed relational databases — AWS handles provisioning, patching, backups, and failover. Common engines: **PostgreSQL**, MySQL, MariaDB, and **Aurora** (AWS’s PostgreSQL/MySQL-compatible storage layer).

Part of: [Amazon Web Services](aws.md). App-side SQL: [PostgreSQL](../../../postgresql/postgresql.md). Observability: [CloudWatch](cloudwatch.md).

---

## Resources

- [Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)
- [Working with DB parameter groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html)
- [DB subnet groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html)

---

## Core Concepts

- **DB instance**  
  A running database server with a chosen **instance class** (vCPU/RAM), **engine + version**, and **storage**. Exposed via an **endpoint** hostname:

  ```text
  mydb.abc123.eu-central-1.rds.amazonaws.com:5432
  ```

- **Engine**  
  Pick PostgreSQL/MySQL/etc. and a version. You manage schema and queries; AWS manages the OS and engine patches (maintenance windows).

- **VPC placement**  
  RDS lives in **private subnets** via a **DB subnet group** (subnets in ≥2 AZs). Not internet-facing by default — apps in [EC2](ec2.md), [ECS](ecs.md), or [EKS](eks.md) connect over the VPC.

- **Security group**  
  Inbound rules allow the DB port (e.g. `5432`) from the app security group or subnet CIDR — not `0.0.0.0/0` in production.

- **Credentials**  
  Master user at create time; prefer **Secrets Manager** rotation or SSM Parameter Store for apps instead of hardcoding in task definitions.

- **Storage**  
  **gp3** (general purpose) or **io** volumes; **storage autoscaling** grows disk when free space is low (set max limit).

- **Parameter group**  
  Engine settings (e.g. `max_connections`, `shared_buffers` on PostgreSQL) applied at instance level.

- **Backups**  
  **Automated backups** — daily snapshots + transaction logs for **point-in-time recovery (PITR)** within the retention window. **Manual snapshots** — long-lived copies you trigger yourself.

- **Multi-AZ**  
  Synchronous standby in another AZ — automatic failover on primary failure. For **high availability**, not read scaling (standby is not readable on standard RDS).

- **Read replica**  
  Async copy of the primary — **read scaling** and optional cross-region DR. App sends writes to primary, reads to replica endpoint(s).

- **Aurora**  
  Cluster with shared storage across AZs; **Aurora replicas** for reads, fast failover, and (with Serverless v2) **ACU-based** compute scaling. PostgreSQL-compatible Aurora is a common pick over plain RDS PostgreSQL at scale.

---

## Common deployment patterns

**Single-AZ dev** — small instance, minimal backup retention, public accessibility off. Cheap; no HA.

**Multi-AZ production** — primary + standby, automated backups (7–35 days), private subnets only. Typical backend for an API on ECS/EC2.

**Primary + read replica(s)** — offload reporting, heavy reads, or read-only API paths. Watch **replication lag** on replicas.

**Aurora cluster** — writer + reader endpoints; scale readers horizontally; Serverless v2 for variable load without resizing instance classes manually.

Apps connect with a standard driver/ORM connection string — host = RDS endpoint, TLS recommended (`sslmode=require` for PostgreSQL).

---

## Scaling

RDS scales differently from [EC2](ec2.md) ASGs or [ECS](ecs.md) task count — mostly **vertical** (bigger instance / more ACUs) and **read replicas**, not “add another primary.”

| Approach                  | What scales           | When                                                                                                |
| ------------------------- | --------------------- | --------------------------------------------------------------------------------------------------- |
| **Instance class change** | vCPU, RAM, network    | Sustained high CPU, memory pressure, connection limits                                              |
| **Storage autoscaling**   | Disk size             | Data growth; set `max_allocated_storage` cap                                                        |
| **Read replica(s)**       | Read throughput       | Read-heavy queries, analytics off primary                                                           |
| **Aurora Serverless v2**  | ACU min/max           | Spiky or unpredictable load on Aurora                                                               |
| **Connection pooling**    | Effective connections | Many app tasks/pods — use **RDS Proxy** or PgBouncer so instance `max_connections` is not exhausted |

Vertical changes and failover cause brief downtime or connection blips unless using Multi-AZ failover — plan maintenance windows or use blue/green deployments (RDS feature) for major upgrades.

---

## Monitoring (CloudWatch)

Key RDS metrics in namespace `AWS/RDS`:

| Metric                         | Use                                                |
| ------------------------------ | -------------------------------------------------- |
| `CPUUtilization`               | Scale up instance class or optimize queries        |
| `FreeStorageSpace`             | Storage autoscaling or disk cleanup                |
| `DatabaseConnections`          | Pooling, scale instance, or tune `max_connections` |
| `ReadLatency` / `WriteLatency` | Storage or query problems                          |
| `ReplicaLag`                   | Read replica falling behind primary                |

### Alarm — low free storage

```json
{
  "AlarmName": "rds-mydb-free-storage-low",
  "Namespace": "AWS/RDS",
  "MetricName": "FreeStorageSpace",
  "Dimensions": [{ "Name": "DBInstanceIdentifier", "Value": "mydb" }],
  "Statistic": "Average",
  "Period": 300,
  "EvaluationPeriods": 1,
  "Threshold": 5000000000,
  "ComparisonOperator": "LessThanThreshold",
  "AlarmActions": ["arn:aws:sns:REGION:ACCOUNT_ID:ops-alerts"]
}
```

Threshold is in bytes (~5 GB free).

### Alarm — high CPU

```json
{
  "AlarmName": "rds-mydb-cpu-high",
  "Namespace": "AWS/RDS",
  "MetricName": "CPUUtilization",
  "Dimensions": [{ "Name": "DBInstanceIdentifier", "Value": "mydb" }],
  "Statistic": "Average",
  "Period": 300,
  "EvaluationPeriods": 3,
  "Threshold": 80,
  "ComparisonOperator": "GreaterThanThreshold",
  "AlarmActions": ["arn:aws:sns:REGION:ACCOUNT_ID:ops-alerts"]
}
```

Alert first; scale instance class or fix queries — RDS does not auto-scale CPU like an ASG.

---
