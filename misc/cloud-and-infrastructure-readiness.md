# Cloud & Infrastructure Readiness

## Resources and tools

1. **Docker**
2. **Github actions** — [GitHub Actions](../technologies/github-actions/github-actions.md)
3. **AWS:** Learning centre (AWS Cloud Practitioner course) is a great place to start. See [Amazon Web Services](../technologies/aws/aws.md) service notes for IAM, EC2, ECR, ECS, RDS, EKS, CloudWatch.
4. **DevOps fundamentals**: roadmaps.sh
5. **API Gateway** — edge auth, throttling, routing (hands-on #2)
6. **ALB** — public HTTP entry for ECS/EKS (hands-on #1)
7. **Monitoring:** Prometheus, Grafana, Coralogix, Open Telemetry
8. **Kubernetes** — [Kubernetes](../technologies/kubernetes.md)
9. **Terraform**

---

## Putting the resources into practice

Reference CI pipeline: [ECR push via GitHub Actions (OIDC)](../technologies/github-actions/hands-on/ecr-github-actions-oidc.md).

### First hands-on — deploy to ECS with Github Actions CI/CD pipeline.

**AWS setup (one-time)**

- VPC with public + private subnets (≥2 AZs)
- **IAM:** GitHub OIDC identity provider + role scoped to the repo ([IAM](../technologies/aws/services/iam.md))
- **ECR:** repository for the app image ([ECR](../technologies/aws/services/ecr.md))
- **RDS:** PostgreSQL or Aurora in **private subnets**; security group allows `5432` from the ECS task security group only ([RDS](../technologies/aws/services/rds.md))
- **ECS:** Fargate cluster, task definition (`awslogs` → CloudWatch), service in private subnets ([ECS](../technologies/aws/services/ecs.md))
- **ALB:** internet-facing, target group → ECS service, listener (HTTPS preferred); health check on app path (e.g. `/healthz`)
- Security groups: ALB → app port on tasks; tasks → RDS; no direct public access to tasks or DB

**CI (GitHub Actions)**

- Workflow on push/PR: lint, test
- On push to `main`:
  - Build Docker image, OIDC → AWS, push to ECR (no long-lived access keys)
  - Deploy to ECS: register new **task definition revision** (image tag = git SHA) → **update service**

**Observability & scaling**

- **CloudWatch alarms:** ECS running task count low, ALB 5xx, RDS CPU / free storage ([CloudWatch](../technologies/aws/services/cloudwatch.md))
- **ECS service auto scaling:** target tracking on CPU or `ALBRequestCountPerTarget` (min/max task count)
- **RDS:** storage autoscaling + alarms — scale **instance class** or add **read replicas** when metrics warrant it; not the same as ECS task autoscaling

---

### Second hands-on — API Gateway, edge auth, deeper observability

Builds on #1. Keep the ALB — Gateway sits in front of it.

- HTTP API + **VPC Link** → existing ALB → ECS
- Move authentication to the edge (JWT authorizer, API keys, or Cognito)
- Restrict ALB security group so traffic only comes from VPC Link (no bypassing Gateway)
- Setup Coralogix
- Setup Prometheus & Grafana

---

### Third hands-on — EKS instead of ECS

Builds on #2. Same app and edge pattern; swap the compute platform.

- **EKS** cluster + managed node groups ([EKS](../technologies/aws/services/eks.md))
- Deploy with Helm or manifests; **Ingress** (AWS Load Balancer Controller) creates the ALB
- Keep API Gateway → VPC Link → Ingress ALB from #2
- Pod autoscaling (HPA); node autoscaling (Karpenter or Cluster Autoscaler)
- Deploy job: `kubectl` / Helm instead of ECS service update
- Monitoring, logging, etc. carry over from #1–#2

### Fourth hands-on - Terraform

- Move the infrastructure definition to Terraform.
