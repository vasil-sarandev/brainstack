# ArgoCD & GitOps

#technology

**GitOps** — git holds the desired cluster state, and an agent in the cluster pulls from git and applies it. **Argo CD** is the most widely used GitOps tool for Kubernetes; **Flux** is the main alternative. Both are graduated CNCF projects.

Parent: [Kubernetes](kubernetes.md). Related: [Workloads](workloads.md), [Rollouts](rollouts.md).

---

## Resources

- [Argo CD docs](https://argo-cd.readthedocs.io/)
- [Argo Rollouts docs](https://argoproj.github.io/argo-rollouts/)
- [Flux docs](https://fluxcd.io/flux/)

---

## Push vs pull

```text
Push:  CI  →  helm upgrade  →  cluster
Pull:  CI  →  commit image tag to git  ←  Argo CD (in cluster) watches  →  applies to cluster
```

| | Push (`helm upgrade` from CI) | Pull (Argo CD / Flux) |
| --- | --- | --- |
| **Cluster credentials** | In CI | Only in the cluster |
| **Source of truth** | Last applied release | Git |
| **Manual `kubectl edit`** | Stays until next deploy | Reverted (self-heal) |
| **Rollback** | `helm rollback` | `git revert` |
| **Setup** | Just CI | Run Argo CD |

The [Helm flow](kubernetes.md#helm) in the main note is push. Many teams still deploy that way; GitOps pays off with many services or clusters.

---

## Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-api-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/acme/deploy-config
    targetRevision: main
    path: charts/my-api
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc     # same cluster
    namespace: my-api
  syncPolicy:
    automated:
      prune: true        # delete resources removed from git
      selfHeal: true     # revert manual changes
```

- **Sync status** — `Synced` / `OutOfSync` with git.
- **Health status** — `Healthy` / `Progressing` / `Degraded`.
- Without `automated`, syncs are triggered manually.
- **ApplicationSet** — generates one Application per directory, cluster, or values file.

---

## Deploy flow

1. Merge to `main`
2. CI builds and pushes `my-api:abc1234`
3. CI commits `image.tag: abc1234` to the config repo (or Argo CD Image Updater does it)
4. Argo CD sees `OutOfSync` → renders the chart → applies
5. Deployment does a normal [rolling update](rollouts.md#maxsurge--maxunavailable)

Argo CD renders Helm charts with `helm template` and applies the output itself, so there's no Helm release — `helm list` / `helm rollback` don't apply.

---

## Rollbacks

`git revert` the tag bump. Argo CD syncs the previous version.

- `kubectl rollout undo` gets reverted by self-heal.
- `argocd app rollback` only works with auto-sync off.

---

## Sync waves & hooks

```yaml
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
```

- **Hooks** — `PreSync` / `Sync` / `PostSync` / `SyncFail`. E.g. a migration [Job](workloads.md#job--cronjob) as `PreSync`; if it fails, the sync stops.
- **Sync waves** — `argocd.argoproj.io/sync-wave: "-1"`. Lower waves are applied and healthy first, e.g. ConfigMaps before the Deployment.

---

## Argo Rollouts

Separate project. Adds a **Rollout** CRD (replaces Deployment) with canary and blue/green strategies.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 10
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 10m }
```

- Traffic split by pod count, or exact weights via ALB / Istio / NGINX.
- **AnalysisTemplates** check Prometheus metrics between steps and abort on failure.
- Works with or without Argo CD.

---
