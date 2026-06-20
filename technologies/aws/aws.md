# Amazon Web Services

#technology

## Resources

- **Docs & References**
  - [AWS Course - AWS Cloud Practitioner Essentials](https://skillbuilder.aws/learn/94T2BEN85A/aws-cloud-practitioner-essentials/8D79F3AVR7)

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
  - **AWS RDS (Relational Database Service)** – Managed SQL databases (MySQL, PostgreSQL, etc.) with automated backups and patching.
  - **AWS IAM (Identity and Access Management)** – Manages users, groups, roles, and permissions securely.
  - **AWS EC2 (Elastic Compute Cloud)** – Virtual servers (instances) to run applications in the cloud.
  - **AWS ECR (Elastic Container Registry)** – Managed Docker container image registry for storing, versioning, and sharing images.
  - **AWS ECS (Elastic Container Service)** – Fully managed container orchestration service for running and scaling Docker containers on AWS.
  - **AWS EKS (Elastic Kubernetes Service)** – Managed Kubernetes service for deploying, managing, and scaling containerized applications.
  - **AWS Fargate** – Serverless compute engine for containers; runs ECS (and EKS) tasks without managing EC2 instances.
  - **AWS ALB (Application Load Balancer)** – Layer 7 load balancer that routes HTTP/HTTPS traffic to targets such as EC2 instances, ECS tasks, or IP addresses.
  - **AWS Elastic Beanstalk** – Platform-as-a-Service (PaaS) to deploy and scale web apps automatically (uses EC2, S3, RDS, etc. under the hood).

---

## Hands-ons

- [ECR push via GitHub Actions (OIDC)](ecr-github-actions-oidc.md)