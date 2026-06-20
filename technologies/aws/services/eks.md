# AWS EKS (Elastic Kubernetes Service)

#technology #aws #docker

**EKS** is AWS’s managed **Kubernetes** control plane. AWS runs the API server, etcd, and schedulers; you run **worker nodes** (managed node groups on [EC2](ec2.md), self-managed nodes, or **Fargate** profiles) and deploy workloads with standard K8s objects — Deployments, Services, Ingress, HPA.

Kubernetes fundamentals: [Kubernetes](../../kubernetes.md). Images from [ECR](ecr.md). Container alternative without K8s: [ECS](ecs.md).

Part of: [Amazon Web Services](aws.md).

---

## Resources

- [Amazon EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Managed node groups](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)
- [Cluster Autoscaler on AWS](https://docs.aws.amazon.com/eks/latest/userguide/autoscaling.html)
- [Karpenter](https://karpenter.sh/)
- [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
- [Kubernetes HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

---

## Core Concepts

- **Control plane**  
  Managed by AWS (multi-AZ). You pay for the cluster hour + worker compute. Access via `kubectl` / Helm after `aws eks update-kubeconfig`.

- **Node groups**  
  **Managed node groups** — AWS creates an ASG of EC2 instances, handles AMI updates and draining. Most common worker setup.

- **Fargate profiles**  
  Run selected pods serverlessly (no node management). Per-pod pricing; good for isolated jobs, less common for high-scale stateless APIs than node groups + Cluster Autoscaler/Karpenter.

- **Add-ons**  
  Typical production stack: **VPC CNI** (pod networking), **CoreDNS**, **kube-proxy**, **AWS Load Balancer Controller** (ALB/NLB Ingress), **EBS CSI** (persistent volumes).

- **IRSA (IAM Roles for Service Accounts)**  
  K8s service account → IAM role mapping. Pods assume roles to call AWS APIs and pull from [ECR](ecr.md) without node-wide credentials.

- **Namespaces & Helm**  
  Same as any cluster — one EKS cluster, many releases/namespaces. See [Kubernetes — Helm](../../kubernetes.md#helm).

---

## Typical flow

```text
ECR image  →  Deployment manifest / Helm values  →  Pods on worker nodes
                                                          ↓
                                              Service / Ingress (ALB)
```

---

## Scaling

EKS scaling is **two layers**: pods (application) and nodes (capacity). Both must work together — HPA adding pods onto a full cluster triggers node scaling.

### Layer 1 — Pod scaling (HPA)

**Horizontal Pod Autoscaler** changes **replica count** of a Deployment based on metrics. This is standard Kubernetes, not EKS-specific — see [Kubernetes — Scaling](../../kubernetes.md#scaling). Pod **requests** must be set — HPA CPU % is relative to requests, not limits.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**Prerequisites on EKS:**

- **metrics-server** (or equivalent) for CPU/memory resource metrics.
- Pod **requests** set on containers — HPA CPU/memory utilization is relative to requests, not limits.

**Common metric sources beyond CPU:**

| Signal | Typical setup |
| --- | --- |
| CPU / memory | metrics-server + resource metric HPA |
| Custom app metric | Prometheus + **prometheus-adapter** or **KEDA** |
| Queue lag (SQS, Kafka) | KEDA ScaledObject |
| ALB/request rate | Custom metric via CloudWatch or Prometheus |

Helm charts often expose autoscaling as values (same idea as in [Kubernetes](../../kubernetes.md#replicacount-vs-autoscaling)):

```yaml
replicaCount: 2
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
```

### Layer 2 — Node scaling (cluster capacity)

When pending pods cannot schedule (`Insufficient cpu/memory`), you need **more nodes**.

**Cluster Autoscaler (CA)** — watches pending pods and scales the **managed node group ASG** up/down. Respects ASG min/max and pod disruption budgets. Well understood; can be slow to react and leaves some bin-packing inefficiency.

**Karpenter** — AWS-native provisioner; creates right-sized EC2 instances (including spot) directly from pod requirements, often faster scale-out and better packing than CA. Increasingly common on new clusters.

```text
Traffic ↑  →  HPA adds pods  →  pending pods  →  CA / Karpenter adds nodes
Traffic ↓  →  HPA removes pods  →  underutilized nodes  →  CA / Karpenter removes nodes
```

**Spot + On-Demand mix** — node groups or Karpenter NodePools with spot for stateless workloads; on-demand for baseline or critical pods (taints/tolerations + node selectors). See [Kubernetes — Node selection & spot instances](../../kubernetes.md#node-selection--spot-instances).

### Layer 3 — Ingress / load balancing

Scaling pods only helps if traffic is spread across them. **AWS Load Balancer Controller** creates ALBs from Ingress resources and registers pod IPs as targets. Health checks (`readinessProbe`) prevent routing to starting or broken pods.

### Practical checklist

1. Set **requests/limits** on containers.
2. Enable **HPA** with sensible min/max and metric (CPU or ALB/custom).
3. Install **metrics-server** (and Prometheus adapter or KEDA if needed).
4. Run **Cluster Autoscaler** or **Karpenter** on managed node groups.
5. Configure **Ingress + readiness probes** so new pods receive traffic safely.

---

## EKS vs ECS

| | **EKS** | **ECS** |
| --- | --- | --- |
| **API** | Kubernetes (portable) | AWS ECS API |
| **Scaling** | HPA + CA/Karpenter | Service Auto Scaling + capacity providers |
| **Ecosystem** | Helm, operators, CNCF tools | AWS-native, simpler surface |

Use EKS when the team and tooling are Kubernetes-centric; use [ECS](ecs.md) when you want less control-plane complexity on AWS alone.

---