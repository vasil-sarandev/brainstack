# Kubernetes Workloads

#technology

Controllers that create and manage pods. You declare the desired state, the controller keeps the pods matching it.

Parent: [Kubernetes](kubernetes.md). Updating them: [Rollouts](rollouts.md).

---

## Resources

- [Workloads](https://kubernetes.io/docs/concepts/workloads/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

---

## Overview

| Controller | Runs | Typical use |
| --- | --- | --- |
| **Deployment** | N interchangeable pods | HTTP APIs, consumers, workers |
| **StatefulSet** | N pods with stable names and their own disk | Databases, Kafka, Elasticsearch |
| **DaemonSet** | One pod per node | Log shippers, node exporter, CNI agents |
| **Job** | Pods until they succeed, then stop | Migrations, backfills |
| **CronJob** | A Job on a schedule | Nightly reports, cleanups |

---

## Deployment & ReplicaSet

```text
Deployment (replicas: 3)
  ├── ReplicaSet v1  →  0 pods
  └── ReplicaSet v2  →  3 pods
```

- **ReplicaSet** — keeps N pods of one template running. Replaces dead pods, never updates them.
- **Deployment** — manages ReplicaSets. Each change to the pod template (`spec.template` — image, env, resources) creates a new ReplicaSet; the rollout scales the new one up and the old one down.
- Old ReplicaSets stay at 0 replicas (`revisionHistoryLimit`, default 10). `kubectl rollout undo` scales one back up.
- Changing `replicas` (manually or by HPA) doesn't create a new ReplicaSet.

You don't create ReplicaSets directly.

```bash
kubectl get rs -l app=my-api     # one row per revision
```

---

## ReplicaSet vs StatefulSet

| | Deployment / ReplicaSet | StatefulSet |
| --- | --- | --- |
| **Pod names** | Random (`api-7d9f-x2k4p`), new on restart | Stable (`db-0`, `db-1`) |
| **DNS** | Through the Service only | Per pod via a headless Service: `db-0.db.my-ns.svc.cluster.local` |
| **Storage** | Shared or none | One PVC per pod (`volumeClaimTemplates`), reattached on restart |
| **Start / stop order** | All at once | In order: `db-0` Ready before `db-1` starts; scale-down from the highest |
| **Updates** | New ReplicaSet, `maxSurge` / `maxUnavailable` | One pod at a time, highest first; `partition` for partial updates |

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: db            # headless Service (clusterIP: None)
  replicas: 3
  selector:
    matchLabels: { app: db }
  template:
    metadata:
      labels: { app: db }
    spec:
      containers:
        - name: postgres
          image: postgres:16
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:      # creates data-db-0, data-db-1, data-db-2
    - metadata:
        name: data
      spec:
        accessModes: [ReadWriteOnce]
        resources:
          requests:
            storage: 20Gi
```

- PVCs stay after scale-down or deleting the StatefulSet — delete them manually.
- Replication and failover are still the app's job (or an operator's, e.g. Strimzi for Kafka).
- EBS volumes are tied to one AZ, so the pod can only reschedule in that zone.
- Most teams use managed databases (RDS, MSK) and keep the cluster stateless.

---

## DaemonSet

One pod on every node (or every node matching a `nodeSelector`), added automatically when a node joins. Used for node agents: Fluent Bit, node-exporter, VPC CNI, EBS CSI driver. Usually needs **tolerations** to run on tainted nodes too.

---

## Job & CronJob

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
spec:
  backoffLimit: 3            # retries before failing
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: my-api:abc1234
          command: ["npm", "run", "migrate"]
```

- **CronJob** — creates a Job on a schedule (`schedule: "0 3 * * *"`). `concurrencyPolicy: Forbid` to skip a run while the previous one is still going.
- Migrations usually run as a Job before the new version rolls out — Helm hook or [ArgoCD PreSync hook](argocd.md#sync-waves--hooks).

---
