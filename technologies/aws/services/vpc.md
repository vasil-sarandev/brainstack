# AWS VPC (Virtual Private Cloud)

#technology #aws

**VPC** is your isolated network inside AWS — CIDR blocks, subnets, routing, and gateways. Every regional resource that needs private IP addressing ([EC2](ec2.md), [ECS](ecs.md) tasks, [EKS](eks.md) nodes/pods, [RDS](rds.md)) lives in a VPC.

Part of: [Amazon Web Services](aws.md). Security groups are VPC-scoped; IAM controls API access, not packet flow.

---

## Resources

- [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [VPC with public and private subnets (NAT)](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html)
- [Security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [VPC endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)
- [AWS VPC CNI (EKS)](https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html)

---

## Core Concepts

- **VPC**  
  A regional virtual network with a **CIDR block** (e.g. `10.0.0.0/16`). You define subnets, route tables, and gateways inside it. Default VPC exists per region; production workloads usually use a custom VPC.

- **Subnet**  
  A slice of the VPC CIDR in one **Availability Zone**. **Public subnets** route `0.0.0.0/0` to an **Internet Gateway (IGW)** — resources can get public IPs. **Private subnets** have no direct internet route; outbound traffic goes through a **NAT Gateway** (or NAT instance) in a public subnet.

- **Route table**  
  Per-subnet (or shared) rules: “traffic to `10.0.0.0/16` stays local; `0.0.0.0/0` → IGW or NAT.” Misconfigured routes are a common cause of “can’t reach the internet” or “can’t reach RDS.”

- **Internet Gateway (IGW)**  
  Attached to the VPC; allows bidirectional internet access for resources with public IPs in public subnets. ALBs for public HTTP APIs are typically placed here.  
  **Only crossed once** — the boundary hop between "outside the VPC" and "inside the VPC" (client → ALB, or a task → ECR/CloudWatch). Traffic between resources *inside* the VPC (ALB → ECS task, ECS task → RDS/MSK) never touches the IGW at all — that's pure internal routing + security groups, same whether or not an IGW even exists.  
  **Default VPC** in every region already has one attached, with every subnet's route table pointing `0.0.0.0/0` at it — i.e. every subnet in a default VPC is already "public." Convenient for quick tests (no NAT Gateway needed for outbound internet access), but means there's no private subnet to fall back on without creating one.

- **NAT Gateway**  
  Managed outbound-only internet for private subnets (pull container images, OS updates, external APIs). One NAT per AZ is common for HA; NAT has hourly + data processing cost.

- **Security group (SG)**  
  Stateful firewall at the ENI level — allow inbound/outbound by port and source (CIDR or another SG). Default deny inbound. Referenced everywhere: [EC2](ec2.md) instances, [ECS](ecs.md) tasks, [RDS](rds.md), ALB.

- **Network ACL (NACL)**  
  Stateless subnet-level firewall (optional extra layer). Most teams rely on SGs + private subnets; NACLs for stricter subnet boundaries or deny lists.

- **Elastic Network Interface (ENI)**  
  Virtual NIC with a private IP in a subnet. Each EC2 instance has at least one; ECS `awsvpc` tasks and EKS pods (via VPC CNI) get their own ENIs.

- **VPC endpoints (PrivateLink / Gateway)**  
  Private connectivity to AWS services without traversing the public internet — e.g. **Gateway endpoint** for S3/DynamoDB, **Interface endpoint** for ECR, Secrets Manager, SSM. Reduces NAT traffic and keeps API calls on the AWS network.

---

## Common deployment patterns

**Public + private subnets (≥2 AZs)** — standard for ECS/EKS + RDS:

```text
Internet
    ↓
  IGW
    ↓
Public subnets (ALB, NAT Gateway)
    ↓
Private subnets (ECS tasks / EC2 / EKS nodes, RDS)
```

- ALB in **public** subnets; app tasks and DB in **private** subnets.
- SG rules: ALB → app port on task SG; task SG → RDS port on DB SG; no `0.0.0.0/0` on the database.
- NAT in each AZ (or one shared NAT for dev) so private workloads can reach [ECR](ecr.md) and the internet.

**Single public subnet (dev only)** — EC2 or Fargate with public IPs. Simple and cheap; not for production DB or unprotected app tiers.

**VPC Link (API Gateway)** — HTTP API in API Gateway connects to an existing ALB or NLB inside the VPC via a **VPC Link**. ALB SG restricted so traffic only arrives from the link — edge auth at Gateway, compute stays private.

---

## How other services use the VPC

| Service | VPC usage |
| --- | --- |
| [EC2](ec2.md) | Instance ENI in a subnet; SG controls ports |
| [ECS](ecs.md) | Fargate/EC2 tasks in subnets (`awsvpc`); ALB target group → task private IP |
| [EKS](eks.md) | Cluster + node groups in subnets; **VPC CNI** assigns pod IPs from VPC CIDR |
| [RDS](rds.md) | **DB subnet group** spans private subnets in ≥2 AZs; endpoint is a DNS name in the VPC |

Cross-AZ traffic within the VPC is normal (ALB in two AZs, Multi-AZ RDS, tasks spread across AZs). Design subnet CIDRs with room for growth — resizing a live VPC CIDR is possible but planning upfront avoids pain.

---

## Quick checklist (production ECS/EKS stack)

- Custom VPC, `/16` or larger, **≥2 AZs**
- Public subnets: IGW route, ALB, NAT Gateway(s)
- Private subnets: app compute, RDS DB subnet group
- SGs: least privilege between ALB → app → DB
- RDS and tasks: **no** public IP
- Optional: VPC endpoints for ECR, S3, CloudWatch Logs to cut NAT cost and tighten egress

---
