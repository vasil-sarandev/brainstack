# Kubernetes Rollouts

#technology

How a Deployment replaces old pods with new ones, and the settings that control it. Strategy concepts (rolling, blue/green, canary): [Deployment Strategies](../../topics/infrastructure/deployment-and-release-engineering/assets/deployment-strategies.md).

Parent: [Kubernetes](kubernetes.md). Related: [Workloads](workloads.md).

---

## Resources

- [Rolling update](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)
- [Pod Disruption Budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
- [Topology Spread Constraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/)

---

## Strategy

A rollout starts on any change to `spec.template` (image, env, resources). `kubectl rollout restart` triggers one without changes, e.g. to pick up a rotated Secret.

```yaml
spec:
  replicas: 4
  minReadySeconds: 10            # pod must stay Ready this long to count
  progressDeadlineSeconds: 600   # mark rollout failed after this long without progress
  revisionHistoryLimit: 10       # old ReplicaSets kept for rollback
  strategy:
    type: RollingUpdate          # or Recreate
    rollingUpdate:
      maxSurge: 1                # extra pods above replicas
      maxUnavailable: 0          # pods allowed below replicas
```

- **RollingUpdate** (default) — new pods come up while old ones go down. Both versions serve traffic during the rollout.
- **Recreate** — stop all old pods, then start new ones. Downtime in between. For when two versions can't run together.

Canary and blue/green need [Argo Rollouts](argocd.md#argo-rollouts), Flagger, or a service mesh.

---

## maxSurge & maxUnavailable

- **maxSurge** — how many pods above `replicas` during the rollout. Needs spare node capacity. Percentages round up.
- **maxUnavailable** — how many pods below `replicas`. Less serving capacity during the rollout. Percentages round down.
- Default **25% / 25%**. Can't both be 0.

Example with `replicas: 4`:

| Setting | Pods during rollout | Behaviour |
| --- | --- | --- |
| `maxSurge: 1, maxUnavailable: 0` | 4–5 | 1 new up → Ready → 1 old down. Slowest, full capacity. Common for APIs. |
| `maxSurge: 0, maxUnavailable: 1` | 3–4 | 1 old down → 1 new up. No extra nodes, 75% capacity. |
| `maxSurge: 25%, maxUnavailable: 25%` | 3–5 | Rounds to 1 / 1 — two pods change at a time. |
| `maxSurge: 100%, maxUnavailable: 0` | up to 8 | Full new set first, then old removed. 2× resources briefly. |

Avoid `maxUnavailable > 0` for services running close to their HPA target.

---

## Readiness & stuck rollouts

The rollout only moves on when new pods are **Ready** ([readiness probe](kubernetes.md#liveness--readiness-probes)):

- No readiness probe — pod counts as Ready when the container starts, before the app is up.
- Readiness fails — rollout stalls, old pods keep serving.
- `minReadySeconds` — catches pods that pass once and crash shortly after.

After `progressDeadlineSeconds` with no progress, the Deployment is marked `ProgressDeadlineExceeded`. K8s doesn't roll back on its own — `kubectl rollout status` fails the CI job, `helm upgrade --atomic` rolls back the release.

```bash
kubectl rollout status deployment/my-api       # non-zero exit on failure
kubectl rollout history deployment/my-api
kubectl rollout undo deployment/my-api
kubectl rollout undo deployment/my-api --to-revision=3
kubectl rollout pause deployment/my-api
kubectl rollout resume deployment/my-api
```

With GitOps, roll back in git — see [ArgoCD — Rollbacks](argocd.md#rollbacks). General: [Rollbacks](../../topics/infrastructure/deployment-and-release-engineering/assets/rollbacks.md).

---

## Graceful shutdown

When an old pod is stopped:

1. Pod is removed from Service endpoints, and in parallel:
2. `preStop` hook runs, then the container gets `SIGTERM`
3. `SIGKILL` after `terminationGracePeriodSeconds` (default 30)

Load balancers can still route to the pod for a few seconds after step 1. A short `preStop` sleep plus finishing in-flight requests on SIGTERM avoids dropped requests:

```yaml
spec:
  terminationGracePeriodSeconds: 45
  containers:
    - name: api
      lifecycle:
        preStop:
          sleep:
            seconds: 10
```

---

## PodDisruptionBudgets

Limits how many pods can be evicted at once by voluntary disruptions — node drains, cluster upgrades, Karpenter / Cluster Autoscaler consolidation.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-api
spec:
  minAvailable: 2          # or maxUnavailable: 1
  selector:
    matchLabels:
      app: my-api
```

- Evictions that would break the budget wait and retry.
- Doesn't cover node crashes or Deployment rolling updates.
- `minAvailable` equal to `replicas` blocks node drains forever.

---

## Spreading replicas

Keeps replicas on different nodes / AZs.

```yaml
spec:
  topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: topology.kubernetes.io/zone     # across AZs
      whenUnsatisfiable: ScheduleAnyway            # DoNotSchedule = hard rule
      labelSelector:
        matchLabels: { app: my-api }
    - maxSkew: 1
      topologyKey: kubernetes.io/hostname          # across nodes
      whenUnsatisfiable: ScheduleAnyway
      labelSelector:
        matchLabels: { app: my-api }
```

**Pod anti-affinity** does a similar job (`podAntiAffinity` with `topologyKey: kubernetes.io/hostname`). The hard version (`requiredDuringScheduling…`) means one pod per node, which can stall a rollout when there's no free node for the surge pod.

Spot vs on-demand placement: [Node selection & spot instances](kubernetes.md#node-selection--spot-instances).

---
