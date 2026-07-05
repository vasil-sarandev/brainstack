# Autoscaling

#infrastructure

Adjust **how many copies** of a workload run (or **how many VMs** back them) based on load — without manual deploys. Distinct from [deployment strategies](deployment-strategies.md), which change **version**; autoscaling changes **count**.

Parent: [Deployment & Release Engineering](../deployment-and-release-engineering.md). Service deep dives: [EC2](../../../technologies/aws/services/ec2.md#scaling), [ECS](../../../technologies/aws/services/ecs.md#scaling), [Kubernetes — Scaling](../../../technologies/kubernetes/kubernetes.md#scaling).

---
## Who scales what?

Containers add a wrinkle: you might scale **replicas** (tasks/pods) and **machines** (VMs that run them) separately. Plain EC2 only has machines.

| Platform | Replicas (app copies) | Machines (hosts) |
| --- | --- | --- |
| **EC2** (no orchestrator) | You — processes per VM | **ASG** scales instances |
| **ECS Fargate** | **Service Auto Scaling** → task count | Not your problem — AWS allocates Fargate capacity |
| **ECS on EC2** | **Service Auto Scaling** → task count | **Capacity provider** + **ASG** → EC2 container instances |
| **EKS** (node groups) | **HPA** → pod count | **Cluster Autoscaler** or **Karpenter** → worker nodes |
| **EKS Fargate** | **HPA** → pod count | Not your problem — Fargate profiles |

```text
More load on ECS Fargate:  Service Auto Scaling adds tasks
More load on ECS + EC2:    Service Auto Scaling adds tasks
                           → if tasks can't fit, capacity provider adds EC2s
More load on EKS:           HPA adds pods
                           → if pods stay Pending, node autoscaler adds nodes
```

**EC2 without an orchestrator** is one row only: ASG changes instance count. How many app processes run on each box is up to you.

---

## EC2 — Auto Scaling Group (ASG)

Scales **whole instances**, not processes inside them.

- **ASG** wraps a launch template (AMI, type, SG, user data) and maintains **min / desired / max** count across AZs
- **Target tracking** policies keep a metric near a target — common signals:
  - `ASGAverageCPUUtilization`
  - `ALBRequestCountPerTarget` (often better for HTTP APIs)
- **Health checks** — ASG replaces unhealthy instances (ELB or EC2 checks)

Typical stack: `ALB → target group → instances in ASG`.

Step scaling and scheduled scaling cover queue depth spikes or known traffic windows.

---

## ECS — Service Auto Scaling

Scales **task count** for a service via **Application Auto Scaling** on `DesiredCount`.

| Metric | When |
| --- | --- |
| `ECSServiceAverageCPUUtilization` | CPU-bound APIs |
| `ECSServiceAverageMemoryUtilization` | Memory-bound work |
| `ALBRequestCountPerTarget` | HTTP behind ALB — request-driven load |

Set **min / max** task count. Target tracking example: “keep average CPU at 70%” or “~1000 ALB requests per target per minute.”

**EC2 launch type only:** if tasks cannot be placed (`Insufficient CPU/Memory`), scale **container instances** via a **capacity provider** linked to an ASG — separate from service task scaling.

**Deploy vs scale:** rolling deploys change image/config; autoscaling changes replica count. Same service controller, different triggers.

**Kafka consumers:** max useful tasks ≈ **partition count** for the topic — scaling past that adds idle consumers.

---

## EKS — HPA and node scaling

### Horizontal Pod Autoscaler (HPA)

Scales **pod count** for a Deployment — overrides static `replicaCount` at runtime.

Common signals:

| Signal | Use |
| --- | --- |
| CPU % (avg across pods) | Default for stateless HTTP |
| Memory | Memory-bound workloads |
| Custom / Prometheus metrics | Queue depth, active workers, **Kafka consumer lag** |
| `ALBRequestCount` (via metrics adapter) | Request-driven APIs |

Floor = Deployment `replicaCount` (or HPA `minReplicas`); ceiling = `maxReplicas`.

### Cluster Autoscaler / Karpenter

When pods are **Pending** (no node has enough CPU/RAM), scale **worker nodes**:

- **Cluster Autoscaler** — adjusts managed node group ASG size
- **Karpenter** — provisions nodes matched to pending pod requirements (often faster/right-sized)

HPA adds pods; node autoscaler adds machines. Both may be needed under load.

**IRSA / resources:** HPA needs accurate CPU if using CPU metrics — set pod `requests`/`limits` sensibly; requests drive scheduling.

---

## Cross-platform comparison

| | **EC2 (ASG)** | **ECS** | **EKS** |
| --- | --- | --- | --- |
| Unit scaled | Instance | Task | Pod |
| Primary API | ASG scaling policies | Application Auto Scaling | HPA |
| HTTP signal | ALBRequestCountPerTarget | ALBRequestCountPerTarget | Custom metric / adapter |
| Host machines | (same as unit — ASG) | Capacity provider + ASG (EC2 launch only) | Cluster Autoscaler / Karpenter |
| Stateless assumption | One app per instance typical | Yes | Yes |

---

## Practical notes

- **Scale stateless tiers** — sessions and sticky data need ALB stickiness or external store (Redis, DB).
- **Cooldowns** — scale-out fast, scale-in slower to avoid flapping (`scale_in_cooldown` / HPA stabilization).
- **Min > 0 in prod** — at least two replicas across AZs for HA on APIs.
- **Alarms alongside autoscaling** — low running task/pod count, ALB 5xx, queue lag — [CloudWatch](../../../technologies/aws/services/cloudwatch.md).

---
