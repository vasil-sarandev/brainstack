# AWS CloudWatch

#technology #aws

**CloudWatch** is AWS’s observability layer: **metrics**, **logs**, and **alarms** for almost every service. It is how you see what workloads are doing and trigger reactions — EC2 ASG scaling, ECS service auto scaling, on-call pages, and dashboards all lean on it.

Part of: [Amazon Web Services](aws.md).

---

## Resources

- [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
- [CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [Container Insights (ECS/EKS)](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html)
- [Embedded Metric Format](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Embedded_Metric_Format.html)

---

## Core Concepts

- **Metrics**  
  Time-series data points (namespace, metric name, dimensions, timestamp, value). AWS services publish many automatically (`AWS/EC2`, `AWS/ECS`, `AWS/ApplicationELB`). You can publish **custom metrics** from app code.

- **Namespaces & dimensions**  
  Metrics are grouped by **namespace** and filtered by **dimensions** (e.g. `ClusterName`, `ServiceName` for ECS). Alarms and dashboards query specific dimension combinations.

- **Logs (CloudWatch Logs)**  
  Central log storage. Common sources: [ECS](ecs.md) `awslogs` driver (`/ecs/myapp`), Lambda, EC2 agent, app SDK. Structure: **log group** → **log streams** → log events.

- **Retention**  
  Log groups have a retention period (days → never expire). Set explicitly — default “never expire” gets expensive.

- **Alarms**  
  Watch a metric (or Logs Insights metric filter) and fire when a threshold is breached for N periods. Actions: SNS notification, Auto Scaling policy, ECS step scaling, etc.

- **Dashboards**  
  Console graphs combining metrics across services — quick operational view, not a full APM replacement.

- **Container Insights**  
  Optional enhanced metrics/logs for [ECS](ecs.md) and [EKS](eks.md) — cluster, service, task, pod-level CPU/memory/network.

---

## Common patterns

**Application logs** — ECS task definition `awslogs` driver → log group `/ecs/<service>`. Search with Logs Insights; correlate by `taskId` or request ID in the log line.

**HTTP service health** — ALB metrics in `AWS/ApplicationELB`: `TargetResponseTime`, `HTTPCode_Target_5XX_Count`, `RequestCountPerTarget`. Often better scaling signals than raw EC2 CPU.

**Scaling trigger** — target tracking policies (see [EC2](ec2.md), [ECS](ecs.md)) use built-in predefined metrics. **Step scaling** and custom cases use explicit CloudWatch alarms.

**Alerting** — alarm → SNS topic → email, Slack webhook, PagerDuty. One alarm per symptom you care about; avoid alert fatigue.

---

## Alarms (examples)

### EC2 ASG — high CPU (step scaling)

```json
{
  "AlarmName": "api-asg-cpu-high",
  "Namespace": "AWS/EC2",
  "MetricName": "CPUUtilization",
  "Dimensions": [{ "Name": "AutoScalingGroupName", "Value": "api" }],
  "Statistic": "Average",
  "Period": 60,
  "EvaluationPeriods": 2,
  "Threshold": 70,
  "ComparisonOperator": "GreaterThanThreshold",
  "AlarmActions": [
    "arn:aws:autoscaling:REGION:ACCOUNT_ID:scalingPolicy:POLICY_ID"
  ]
}
```

Target tracking (preferred for steady-state) hides this — you set `target_value` on the ASG policy instead. Step alarms like above are for custom thresholds.

### ECS service — running task count low

```json
{
  "AlarmName": "api-ecs-running-tasks-low",
  "Namespace": "AWS/ECS",
  "MetricName": "RunningTaskCount",
  "Dimensions": [
    { "Name": "ClusterName", "Value": "my-cluster" },
    { "Name": "ServiceName", "Value": "api" }
  ],
  "Statistic": "Average",
  "Period": 60,
  "EvaluationPeriods": 3,
  "Threshold": 1,
  "ComparisonOperator": "LessThanThreshold",
  "TreatMissingData": "breaching",
  "AlarmActions": ["arn:aws:sns:REGION:ACCOUNT_ID:ops-alerts"]
}
```

Useful as a **health alert** (tasks crashing) separate from auto scaling.

### ALB — 5xx spike

```json
{
  "AlarmName": "api-alb-5xx",
  "Namespace": "AWS/ApplicationELB",
  "MetricName": "HTTPCode_Target_5XX_Count",
  "Dimensions": [
    { "Name": "LoadBalancer", "Value": "app/my-alb/abc123" },
    { "Name": "TargetGroup", "Value": "targetgroup/api/xyz789" }
  ],
  "Statistic": "Sum",
  "Period": 300,
  "EvaluationPeriods": 1,
  "Threshold": 10,
  "ComparisonOperator": "GreaterThanThreshold",
  "AlarmActions": ["arn:aws:sns:REGION:ACCOUNT_ID:ops-alerts"]
}
```

### Logs Insights — query (ad hoc debugging)

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 50
```

Run against a log group (e.g. `/ecs/api`) in the console or via API.

---
