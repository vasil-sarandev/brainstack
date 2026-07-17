# Deploy to ECS

#infrastructure

AWS-native **container** orchestration: task definitions, services, Fargate. Typical path for Dockerized APIs and workers when you do not want to operate Kubernetes.

Parent: [Deployment targets](deployment-targets.md). Service details: [AWS ECS](../../../technologies/aws/services/ecs.md), [ECR](../../../technologies/aws/services/ecr.md).

Case study: [Node-distributed-monolith on AWS ECS](ecs-node-distributed-monolith.md)

---

## Pipeline shape

```text
CI: docker build → push to ECR (:git-sha)
  → register task definition revision (new image tag)
  → ecs update-service --force-new-deployment
```

ECR is the artifact store. **Push ≠ deploy** — ECS must reference the new tag and roll out tasks.

---

## Core objects

| Object | Role |
| --- | --- |
| **Task definition** | Image URI, CPU/RAM, env, secrets, command, log driver |
| **Task** | One running instance of a task definition |
| **Service** | Keeps N tasks up, rolling deploys, optional ALB registration |
| **Cluster** | Logical grouping (Fargate or EC2 capacity) |

Public HTTP: **ALB** → target group → tasks in private subnets.

---

## One image, many services

A single ECR tag can back multiple ECS services with different `command` overrides — API vs background workers vs Kafka consumers. Each service deploys and scales independently.

See [node-distributed-monolith case study](../case-studies/node-distributed-monolith.md).

---

## Config and CI auth

- Runtime env in task definition; secrets from SSM / Secrets Manager
- **Task execution role** — pull ECR, write logs
- **Task role** — app calls AWS APIs at runtime
- CI: GitHub **OIDC** → IAM role (`ecr:*` push, later `ecs:UpdateService`) — [ECR push via GitHub Actions (OIDC)](../../../technologies/github-actions/hands-on/ecr-github-actions-oidc.md)

---

## ECS vs EKS

| ECS | EKS |
| --- | --- |
| Task definition + service | Deployment + Service + Ingress |
| AWS APIs, Fargate-first | Kubernetes manifests / Helm |
| Less operational surface | Portable K8s ecosystem |

Same Docker image and ECR push; different deploy config format. See [Deploy to EKS](eks.md).
