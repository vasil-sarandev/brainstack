# Traffic Routing & Service Discovery

#infrastructure

How external traffic finds **which specific instance** of a workload to hit, and how that answer stays correct as instances start, stop, and move. Distinct from [autoscaling](autoscaling.md), which changes **how many copies** exist; this is about **routing to whichever copies currently exist and are healthy**.

Parent: [Deployment & Release Engineering](../deployment-and-release-engineering.md). Service deep dives: [ALB & Target Groups](../../../../technologies/aws/services/alb.md), [ECS](../../../../technologies/aws/services/ecs.md), [Kubernetes](../../../../technologies/kubernetes/kubernetes.md).

---

## The shape is the same everywhere

Every platform splits this into two layers: a **stable entry point** (fixed address, does routing/load balancing) and a **membership registry** (tracks which backend IPs currently exist and are healthy). Instances are ephemeral — the entry point is what stays constant across restarts/redeploys.

```text
Client → [stable entry point] → [membership registry] → actual instance
```

| Platform | Stable entry point | Membership registry |
| --- | --- | --- |
| **EC2 (ASG) / ECS** | ALB (listener + rules) | Target group |
| **Kubernetes / EKS** | Ingress + Ingress Controller | Service |

---

## ALB & target groups (EC2, ECS)

- **ALB** — accepts the connection, optionally terminates TLS, evaluates **listener rules** to pick a target group, then load-balances across that group's currently-healthy targets. Its own DNS name is the fixed address clients use; internet-facing or internal is a choice made at creation, not an inherent property.
- **Target group** — passive registry. It does **not** discover targets itself; ECS (or the ASG's ELB integration) actively registers/deregisters instance or task IPs into it as they start and stop. It also owns **health checking** — runs on its own schedule, independent of the ALB — and only currently-healthy members receive traffic.
- **Routing vs load balancing are different jobs**: routing (which target group) happens via listener rules (path/host-based, e.g. `/api/v1/*` → group A, `/api/v2/*` → group B); load balancing (which specific target within that group) happens after, round-robin/least-outstanding-requests across whatever's currently healthy.
- **Deregistration delay** (default 300s) — an old target isn't considered fully gone until this drains, which is usually why a deployment looks "done" in the console well before it's officially "stable."

Full detail: [ALB & Target Groups](../../../../technologies/aws/services/alb.md).

---

## Ingress & Services (Kubernetes / EKS)

- **Ingress** — a Kubernetes API object declaring HTTP/HTTPS routing rules (path/host-based), same idea as ALB listener rules. By itself it does nothing — Kubernetes ships the API, not an implementation.
- **Ingress Controller** — the thing that actually watches Ingress resources and implements them. Not included by default; you install one. On EKS, the standard choice is the **AWS Load Balancer Controller**, which reacts to Ingress resources by provisioning a real ALB behind the scenes — same AWS machinery as the row above, just driven by YAML instead of console clicks.
- **Service** — Kubernetes' membership registry, conceptually identical to a target group: tracks which pod IPs currently back it, and load-balances across them (via `kube-proxy`, typically iptables/IPVS rules). A `ClusterIP` Service is internal-only; Ingress routes to a Service, which then load-balances across the pods behind it.
- **Readiness probe** — the Kubernetes equivalent of a target group health check, run per-pod by the kubelet; only ready pods receive traffic.

---

## Direct concept mapping

| ECS / ALB                               | Kubernetes                                         |
| --------------------------------------- | -------------------------------------------------- |
| ALB + listener rules                    | Ingress resource + Ingress Controller              |
| Target group                            | Service                                            |
| ECS task                                | Pod                                                |
| ECS registers task IP into target group | kube-proxy registers pod IP into Service endpoints |
| Target group health check               | Pod readiness probe                                |

---

## Beyond simple routing

Both stacks support more than "one rule, one destination":

- **Host/path-based routing** — one ALB (or one Ingress) serving multiple services/versions behind different paths or hostnames, instead of one load balancer per service.
- **Weighted / canary routing** — split traffic by percentage across two target groups (or two Ingress backends), shifting weight as confidence in a new version builds — a lighter-weight alternative to a full blue/green cutover. See [Deployment Strategies](deployment-strategies.md).
- **TLS termination** — the entry point (ALB or Ingress Controller) decrypts HTTPS and forwards plain HTTP internally, so individual instances/pods never handle certs themselves.

---
