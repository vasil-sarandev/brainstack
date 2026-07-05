# Deploy to EKS

#infrastructure

Run containers on **managed Kubernetes** (AWS EKS). Workloads are declared as Deployments, Services, Ingress, and optionally Helm releases — same model as any K8s cluster.

Parent: [Deployment targets](deployment-targets.md). Service details: [AWS EKS](../../../technologies/aws/services/eks.md), [Kubernetes](../../../technologies/kubernetes/kubernetes.md).

---

## Pipeline shape

```text
CI: docker build → push to ECR (:git-sha)
  → update image tag in manifest / Helm values
  → kubectl apply  OR  helm upgrade  OR  ArgoCD sync
```

GitOps (ArgoCD, Flux) watches the repo and reconciles cluster state — deploy config often lives beside the app or in an infra repo.

---

## Core objects (vocabulary)

| K8s | Role | ECS equivalent |
| --- | --- | --- |
| **Pod** | Running container(s) | Task |
| **Deployment** | Desired replica count, rolling updates | ECS Service + task definition |
| **Service** | Stable cluster DNS / load balancing to pods | Target group / Cloud Map |
| **Ingress** | External HTTP(S) — often creates ALB via controller | ALB + listener rules |
| **HPA** | Autoscale pod count on CPU or custom metrics | ECS Service Auto Scaling |

One **Deployment** (or Helm release) per independently deployable process. Same ECR image, different `command` or values file per Deployment.

---

## AWS-specific pieces

- **Managed node groups** or **Fargate profiles** for worker capacity
- **AWS Load Balancer Controller** — Ingress → ALB
- **IRSA** — pod service account maps to IAM role (ECR pull, AWS API access)
- **Cluster Autoscaler / Karpenter** — scale nodes when pods cannot be scheduled

---

## Config layout (typical)

```text
my-api/
  k8s/
    deployment.yaml
    service.yaml
    ingress.yaml
  values-prod.yaml      # Helm
```

Or a shared Helm chart + per-service values files; CI iterates and runs `helm upgrade` per release.

---

## When to choose ECS instead

Fewer moving parts on AWS-only stacks, no control plane or Helm to learn, Fargate without node management. Choose EKS when you need the K8s ecosystem (operators, portable manifests, existing team tooling).

See [Deploy to ECS](ecs.md).
