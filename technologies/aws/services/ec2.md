# AWS EC2 (Elastic Compute Cloud)

#technology #aws

**EC2** provides resizable virtual servers (**instances**) in AWS. It is the default compute layer underneath many other services — ECS container instances, EKS worker nodes, bastion hosts, and single-VM app deploys (Docker on EC2).

Part of: [Amazon Web Services](aws.md). Often paired with [ECS](ecs.md), [EKS](eks.md), and images from [ECR](ecr.md).

---

## Resources

- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [EC2 instance types](https://aws.amazon.com/ec2/instance-types/)
- [Auto Scaling groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html)
- [Launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html)

---

## Core Concepts

- **Instance**  
  A VM with a chosen **instance type** (vCPU, RAM, network), **AMI** (disk image), and **security groups** (firewall rules). Lives in a **subnet** inside a VPC.

- **AMI (Amazon Machine Image)**  
  Template for boot volume — OS, packages, optional baked-in app. You can build custom AMIs (e.g. Packer) or use AWS-provided ones.

- **EBS volumes**  
  Persistent block storage attached to an instance. Root volume + optional data volumes; snapshot to S3 for backups and new AMIs.

- **Security groups**  
  Stateful firewall at the ENI level — allow inbound/outbound by port and source (CIDR, other SG). Default: no inbound from internet until you open it.

- **Key pair / IAM instance profile**  
  SSH access (Linux) via key pair; **instance profile** attaches an IAM role so the VM can call AWS APIs (SSM, ECR pull, S3) without static keys.

- **Elastic IP**  
  Static public IPv4 you can reattach after instance replacement. Useful for simple setups; prefer ALB + ASG for production HTTP.

- **User data**  
  Script run at first boot — install Docker, pull from [ECR](ecr.md), register with an orchestrator, etc.

---

## Common deployment patterns

**Single instance** — one EC2, Docker Compose or bare process, Elastic IP or Route 53 A record. Simplest; no auto-healing.

**ASG + ALB** — instances behind an **Application Load Balancer**, Auto Scaling Group replaces unhealthy nodes and scales count. Standard for stateless web/API tiers on EC2.

**Worker / bastion** — fixed-size ASG (min = max = 1) or standalone instance for jobs, SSH jump host, or deploy target.

---

## Scaling

EC2 scaling is **instance count**, not process count inside the VM. You scale by adding or removing whole VMs.

### Auto Scaling Group (ASG)

An ASG wraps a **launch template** (AMI, instance type, SG, user data, IAM profile) and maintains a **desired capacity** across subnets/AZs.

```text
ALB (HTTP) → Target Group → EC2 instances in ASG
                              ↑
                    Scaling policies adjust desired capacity
```

- **Minimum / desired / maximum** — floor, target, and ceiling instance count.
- **Health checks** — ASG marks instances unhealthy using ELB health checks or EC2 status checks; replaces them automatically.
- **Multi-AZ** — spread instances across availability zones for fault tolerance.

### Scaling policies (common)

**Target tracking** — most common for web workloads. ASG adjusts count to keep a metric near a target:

| Metric | Typical use |
| --- | --- |
| `ASGAverageCPUUtilization` | General compute-bound services |
| `ALBRequestCountPerTarget` | HTTP APIs behind an ALB (often better than CPU alone) |

Example policy intent: “Keep average CPU at 60%” or “~1000 requests per instance per minute.”

**CPU target tracking:**

```hcl
resource "aws_autoscaling_policy" "api_cpu" {
  name                   = "api-cpu-target"
  autoscaling_group_name = aws_autoscaling_group.api.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 60.0
  }
}
```

**ALB request count per target** (often better for HTTP APIs):

```hcl
resource "aws_autoscaling_policy" "api_requests" {
  name                   = "api-alb-requests"
  autoscaling_group_name = aws_autoscaling_group.api.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ALBRequestCountPerTarget"
      resource_label           = "${aws_lb.api.arn_suffix}/${aws_lb_target_group.api.arn_suffix}"
    }
    target_value = 1000.0   # requests per instance per minute
  }
}
```

**Scheduled scaling** — predictable traffic (e.g. scale up weekdays 08:00):

```hcl
resource "aws_autoscaling_schedule" "scale_up_morning" {
  scheduled_action_name  = "scale-up-weekdays"
  autoscaling_group_name = aws_autoscaling_group.api.name
  min_size               = 4
  max_size               = 20
  desired_capacity       = 8
  recurrence             = "0 8 * * MON-FRI"
  time_zone              = "Europe/Sofia"
}
```

**Step scaling / simple scaling** — add N instances when a CloudWatch alarm breaches a threshold. More manual tuning; target tracking is usually preferred.

### Launch template updates

Rolling deploys on EC2 often use a **new launch template version** + ASG **instance refresh** (replace instances gradually with the new AMI/config) rather than SSH-and-patch.

### Spot vs On-Demand

**Spot instances** — much cheaper; AWS can reclaim with notice. Fine for fault-tolerant, stateless ASG workloads with mixed On-Demand base capacity.

**On-Demand / Reserved** — stable baseline; Reserved/Savings Plans for predictable steady load.

### What EC2 scaling does *not* do

Scaling the ASG adds VMs; it does not automatically scale processes **inside** a box. For container/worker density per host, use [ECS](ecs.md) or [EKS](eks.md) task/pod scaling on top of EC2 (or Fargate).

---
