# Infrastructure Hands On

#infrastructure

Learning roadmap - hands-on projects to go from "infra team handles it" to owning build → ship → run. Reference concepts live under [Deployment & Release Engineering](deployment-and-release-engineering.md).

---
## Resources and tools

1. **Docker** — [Docker](../technologies/docker.md)
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

### First hands-on — deploy to ECS with Github Actions CI/CD pipeline.

GitHub Actions builds the Docker image and pushes it to ECR on merge to `main`. ECS Fargate pulls the image from ECR and runs the API. SSM Parameter Store (SecureString) supplies secrets at task startup (RDS host, password, JWT, Stripe keys, etc.) as environment variables. RDS PostgreSQL in the same VPC, not publicly accessible. The RDS security group allows inbound 5432 only from the ECS task security group (and temporarily from my IP for local migrations). An Application Load Balancer (ALB) is the public entry point for HTTP(S). It receives traffic from the internet, performs health checks, and forwards requests to the ECS service. The ECS security group only accepts application traffic from the ALB, so tasks aren't exposed directly to the internet.

Flow: `Client → ALB → ECS → RDS`

**AWS setup (one-time)**

- **VPC**: Default is pretty good for a starter point.
	- **Security groups used for RDS/ECS**: Allow inbound traffic from personal IP + ECS only; Allow all outbound traffic; Disable all public incoming traffic.
- **IAM:** GitHub OIDC identity provider + role scoped to the repo ([IAM](../technologies/aws/services/iam.md)); Task execution Role for ECS.
- **ECR:** repository for the app image ([Reference for ECR-Github Actions hands-on](../technologies/github-actions/hands-on/ecr-github-actions-oidc.md))
- **RDS:** PostgreSQL
- **ECS:** Fargate cluster, task definition (`awslogs` → CloudWatch), service in private subnets ([ECS](../technologies/aws/services/ecs.md))
- **ALB:** internet-facing, target group → ECS service, listener (HTTPS preferred); health check on app path (e.g. `/healthz`)

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

---
