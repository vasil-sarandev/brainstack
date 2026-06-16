
# Kubernetes (K8s)

#technology 

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It turns a collection of individual servers (nodes) into a single, unified computing resource that self-heals and scales based on demand.

## Resources

- [Kubernetes Documentation: Concepts](https://kubernetes.io/docs/concepts/)
- [K8s Interactive Tutorials](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Helm: The K8s Package Manager](https://helm.sh/)

## Core Concepts

- **Cluster & Nodes:** A cluster consists of a **Control Plane** (the brain) and multiple **Worker Nodes** (the muscle—usually VPSes or bare metal). A cluster is the entire environment — all nodes, all pods, all services.
- **Pods:** The smallest deployable unit. It’s a logical wrapper around one or more containers that share the same network and storage.
- **Deployment:** The controller that says "run N copies of this pod, keep them alive, roll out updates". This is the logical unit grouping pods with a purpose — not the cluster.
- **Desired State:** You don’t tell K8s _how_ to start a server; you give it a **YAML manifest** describing what you want (e.g., "3 replicas of my API"), and K8s makes it happen.
- **Self-Healing:** If a container crashes or a node dies, K8s automatically restarts or reschedules those pods on healthy hardware without human intervention.
- **Service Discovery & Load Balancing:** K8s gives a stable DNS name and IP to a group of pods, automatically balancing traffic across them regardless of which node they live on.
- **Horizontal Scaling:** Automatically increases or decreases the number of pods based on CPU/RAM usage or custom metrics.
- **Declarative Updates:** Supports "Rolling Updates" where new versions are deployed one by one, ensuring zero downtime during releases.
- **Bin Packing:** Intelligently places containers on nodes to maximize resource utilization and minimize infrastructure costs. Multiple pods from completely different deployments share the same node — K8s schedules them based on available CPU/memory.

---

## Scaling

### replicaCount vs autoscaling

`replicaCount` in a deployment YAML sets the number of pods. For a fixed-size deployment this is all you need. For dynamic scaling you add an **HPA (HorizontalPodAutoscaler)** on top, which overrides `replicaCount` at runtime.

```yaml
replicaCount: 1        # starting pod count (also the floor)

autoscaling:
  enabled: true
  maxPods: 20          # ceiling — HPA won’t go above this
  cpu:
    enabled: true
    targetCpuPercent: 70   # scale up when avg CPU across pods hits 70%
```

The HPA continuously watches the metric and adjusts pod count between `replicaCount` and `maxPods`.

### Scaling signals

**CPU-based** — standard for HTTP services. When average CPU across running pods exceeds the threshold, new pods spin up. Simple and works well for compute-bound workloads.

```yaml
autoscaling:
  cpu:
    enabled: true
    targetCpuPercent: 80
```

**Custom Prometheus metric** — more meaningful for specific workloads. For example, an HTTP service might expose a metric representing % of workers actively handling a request, computed via a recording rule from raw gauges the app emits. The HPA watches that instead of (or alongside) raw CPU.

```yaml
autoscaling:
  prometheus:
    enabled: true
    metricName: "loyalty_active_unicorn_percentage"
    targetValue: 50
    prometheusLabels:
      job: "loyalty-unicorn"
```

**Kafka lag-based** — for consumer pods. The HPA watches `kafka_consumergroup_lag_sum` for a specific topic. When unconsumed messages pile up, new pods spin up, join the consumer group, and Kafka rebalances partitions across them.

```yaml
autoscaling:
  prometheus:
    metricName: "kafka_consumergroup_lag_sum"
    prometheusLabels:
      topic: "my-topic"
      consumergroup: "my-consumer-group"
    targetValue: 1000
```

### Resources: requests vs limits

```yaml
resources:
  cpu: 500m        # request — reserved on the node for scheduling
  memory: 1000Mi
  limits:
    cpu: 1500m     # limit — max the container can burst to
    memory: 2000Mi
```

`requests` are what Kubernetes uses when deciding which node to place a pod on — it reserves that much capacity. `limits` are enforced at runtime but don’t affect scheduling. Setting requests too high wastes node capacity even when pods are idle.

---

## Services & Networking

A **Kubernetes Service** is a stable DNS entry + internal load balancer in front of a set of pods. Without it, pods are unreachable by other services (they have ephemeral IPs that change on restart).

```yaml
service:
  create: true
  type: ClusterIP       # internal only — reachable within the cluster
  ports:
    - name: http
      port: 80
      targetPort: 3000
```

- **ClusterIP** — internal only, other pods in the cluster can call it by name
- **Ingress** — punches a hole to the outside world; `ingress: enabled: true` exposes the service externally

Pods that don’t receive traffic (e.g. Kafka consumers, Sidekiq workers) don’t need a Service at all.

### Liveness & Readiness Probes

Only relevant for pods that receive traffic. K8s uses these to know if a pod is healthy enough to route requests to.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 3000
  initialDelaySeconds: 30    # wait before first check
  failureThreshold: 5        # restart pod after 5 consecutive failures

readinessProbe:
  httpGet:
    path: /readyz
    port: 3000
  initialDelaySeconds: 15    # wait before accepting traffic
```

---

## Helm

Helm is the K8s package manager. Instead of writing raw K8s manifests (Deployment YAML, Service YAML, HPA YAML separately), you write a **values file** and Helm merges it with a **chart** (a template) to produce all the manifests and apply them.

```
values.yaml  +  chart template  →  helm upgrade  →  K8s Deployment + Service + HPA
```

A typical flow in CI/CD:
1. Merge to master
2. GitHub Actions calls a shared `helm-deploy.yml` workflow
3. That workflow iterates over all values files with `deployByDefault: true`
4. For each file: `helm upgrade <deploymentName> <chartName> -f <values-file>`
5. Helm renders and applies the K8s objects to the cluster

Each values file becomes one independent Helm release — one Deployment (+ optional HPA, Service, etc.) in the cluster. They all live in the same cluster, same namespace, just isolated by their `deploymentName`.

### Node selection & spot instances

```yaml
nodeSelector:
  node.kubernetes.io/lifecycle: spot   # only schedule on spot EC2 instances
  scaler: castai

tolerations:
  - key: "scaler"
    value: "castai"
    operator: "Equal"
    effect: "NoSchedule"
```

Spot instances are significantly cheaper but can be reclaimed by AWS at any time — K8s handles rescheduling automatically when that happens. Most stateless workloads (consumers, workers, HTTP services) run on spot. The `nodeSelector` ensures a pod only lands on nodes with matching labels.