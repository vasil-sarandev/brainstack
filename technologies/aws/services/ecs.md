# AWS ECS (Elastic Container Service)

#technology #aws #docker

**ECS** is AWS’s native container orchestrator. You define **tasks** (containers + CPU/RAM + env) and **services** (long-running desired count), and ECS schedules them on **Fargate** (serverless) or **EC2** instances you manage.

Part of: [Amazon Web Services](aws.md). Images from [ECR](ecr.md); HTTP traffic usually via ALB. Alternative on AWS: [EKS](eks.md) (Kubernetes).

---

## Resources

- [Amazon ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [Task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)
- [ECS Service Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)
- [Fargate on ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)

---

## Core Concepts

- **Cluster**  
  Logical grouping of capacity — Fargate-only, EC2-only, or mixed via **capacity providers**.

- **Task definition**  
  Blueprint: container image URI ([ECR](ecr.md)), CPU/memory, port mappings, env vars, secrets (SSM/Secrets Manager), log driver (CloudWatch). JSON or Terraform/CDK.

- **Task**  
  One running instance of a task definition (one or more containers). Ephemeral — when it stops, local disk is gone unless using EFS.

- **Service**  
  Keeps **N tasks** running, registers them with a load balancer, replaces failed tasks, supports rolling deployments.

- **Launch types**
  - **Fargate** — no EC2 to manage; pay per task vCPU/RAM. Simplest ops.
  - **EC2** — ECS places tasks on EC2 instances in the cluster; you scale the **container instance ASG** separately from **service task count**.

- **Networking**  
  Tasks get ENIs in VPC subnets (`awsvpc` mode). ALB **target group** routes to task private IPs on the container port.

- **Roles**
  - **Task execution role** — pull from ECR, write logs, fetch secrets at start.
  - **Task role** — credentials the app uses at runtime (S3, DynamoDB, etc.).

---

## Typical flow

```text
ECR image  →  task definition  →  service (desired count: 2)
                                      ↓
                              ALB → target group → tasks
```

CI pushes to [ECR](ecr.md); deploy updates the service to a new task definition revision (new image tag/digest).

---

## Scaling

ECS has two scaling layers — know which one you are adjusting.

### 1. Service Auto Scaling (task count)

Scales **how many tasks** run for a service — the main knob for HTTP APIs and workers.

- Set **desired count** manually, or attach **Application Auto Scaling** to the service.
- **Target tracking** policies (most common):

| Metric | When to use |
| --- | --- |
| `ECSServiceAverageCPUUtilization` | CPU-bound APIs |
| `ECSServiceAverageMemoryUtilization` | Memory-bound workloads |
| `ALBRequestCountPerTarget` | HTTP services behind ALB — often best track for request-driven load |

Example intent: min 2 tasks, max 20, target 70% CPU or 1000 requests/target/minute.

**CPU target tracking:**

```hcl
resource "aws_appautoscaling_target" "api" {
  max_capacity       = 20
  min_capacity       = 2
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.api.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "api_cpu" {
  name               = "api-cpu"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.api.resource_id
  scalable_dimension = aws_appautoscaling_target.api.scalable_dimension
  service_namespace  = aws_appautoscaling_target.api.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value       = 70.0
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}
```

**ALB request count per target:**

```hcl
resource "aws_appautoscaling_policy" "api_requests" {
  name               = "api-alb-requests"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.api.resource_id
  scalable_dimension = aws_appautoscaling_target.api.scalable_dimension
  service_namespace  = aws_appautoscaling_target.api.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ALBRequestCountPerTarget"
      resource_label         = "${aws_lb.api.arn_suffix}/${aws_lb_target_group.api.arn_suffix}"
    }
    target_value = 1000.0
  }
}
```

- **Step scaling** — scale on CloudWatch alarms (queue depth, custom metric).
- **Scheduled scaling** — known traffic windows (scale up before peak).

Service scaling assumes tasks are **stateless** and safe to add/remove; sessions/sticky state need ALB stickiness or external store.

### 2. Cluster capacity (EC2 launch type only)

On **Fargate**, AWS adds capacity automatically — you only scale task count.

On **EC2 launch type**, tasks need free CPU/RAM on container instances:

- **Container instance ASG** — scale EC2 instances when tasks cannot be placed (`Insufficient CPU/Memory` events).
- **Capacity providers** — link an ASG to the cluster; ECS scales the ASG when tasks cannot be placed. **Managed scaling** rules:

```hcl
managed_scaling {
  status                    = "ENABLED"
  target_capacity           = 80    # cluster ~80% utilized before scaling out
  minimum_scaling_step_size = 1
  maximum_scaling_step_size = 10
}
```

```text
Service Auto Scaling     →  more/fewer tasks
Capacity provider / ASG  →  more/fewer EC2 hosts (EC2 launch type only)
```

### Deployments vs scaling

- **Rolling update** — `minimumHealthyPercent` / `maximumPercent` control how many tasks stay up during a new task definition rollout.
- Scaling changes **replica count**; deployments change **image/config** — both use the same service controller.

### Fargate vs EC2 scaling trade-off

| | **Fargate** | **EC2** |
| --- | --- | --- |
| **Task scaling** | Application Auto Scaling on service | Same |
| **Host scaling** | Not your problem | ASG + capacity provider |
| **Ops** | Lower | Patch AMIs, instance density, bin-packing |

---

## ECS vs EKS

Choose **ECS** when you want AWS-native APIs, fewer moving parts, and no Kubernetes operational surface. Choose **[EKS](eks.md)** when you need K8s ecosystem (Helm charts, operators, HPA/custom metrics patterns from [Kubernetes](../../kubernetes.md)).

---
