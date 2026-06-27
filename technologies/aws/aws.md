# Amazon Web Services

#technology

## Resources

- **Docs & References**
  - [AWS Course - AWS Cloud Practitioner Essentials](https://skillbuilder.aws/learn/94T2BEN85A/aws-cloud-practitioner-essentials/8D79F3AVR7)
- **Services**
	- [AWS IAM (Identity and Access Management)](iam.md)
	- [AWS EC2 (Elastic Compute Cloud)](ec2.md)
	- [AWS ECR (Elastic Container Registry)](ecr.md)
	- [AWS ECS (Elastic Container Service)](ecs.md)
	- [AWS EKS (Elastic Kubernetes Service)](eks.md)
	- [AWS CloudWatch](cloudwatch.md)
	- [AWS RDS (Relational Database Service)](rds.md)

---

## Core Concepts

- **Cloud Computing**  
   On-demand delivery of IT resources over the Internet with pay-as-you-go pricing.
- **Shared Responsibility Model**
  - AWS → _Security of the Cloud_ (infrastructure, hardware, networking)
  - Customer → _Security in the Cloud_ (OS, applications, data, IAM)
- **Regions and Availability Zones (AZs)**  
   AWS Regions are geographic areas with multiple isolated AZs (data centers) for high availability and fault tolerance.
- **Core AWS Services**
  - **AWS S3 (Simple Storage Service)** – Object storage for any amount of data; great for backups, static website hosting, and data lakes.
  - **[AWS RDS (Relational Database Service)](rds.md)** – Managed PostgreSQL, MySQL, and Aurora; backups, Multi-AZ, read replicas.
  - **[AWS IAM (Identity and Access Management)](iam.md)** – Users, groups, roles, and policies; controls access to every AWS API.
  - **[AWS EC2 (Elastic Compute Cloud)](ec2.md)** – Virtual servers (instances); foundation for single-VM apps, ECS EC2 capacity, and EKS worker nodes.
  - **[AWS ECR (Elastic Container Registry)](ecr.md)** – Managed Docker container image registry; typical target for CI-built images before ECS/EKS/Fargate.
  - **[AWS ECS (Elastic Container Service)](ecs.md)** – AWS-native container orchestration (Fargate or EC2); service-level task scaling + capacity providers.
  - **[AWS EKS (Elastic Kubernetes Service)](eks.md)** – Managed Kubernetes control plane; HPA for pods, Cluster Autoscaler/Karpenter for nodes.
  - **[AWS CloudWatch](cloudwatch.md)** – Metrics, logs, and alarms; default observability and scaling signal source for EC2, ECS, ALB, and custom apps.
  - **AWS Fargate** – Serverless compute engine for containers; runs ECS (and EKS) tasks without managing EC2 instances.
  - **AWS ALB (Application Load Balancer)** – Layer 7 load balancer that routes HTTP/HTTPS traffic to targets such as EC2 instances, ECS tasks, or IP addresses.
  - **AWS Elastic Beanstalk** – Platform-as-a-Service (PaaS) to deploy and scale web apps automatically (uses EC2, S3, RDS, etc. under the hood).

---
