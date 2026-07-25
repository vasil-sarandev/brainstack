# Prometheus & Grafana

#technology

Prometheus is an open-source **metrics collection and alerting** system built for pull-based, time-series monitoring. Grafana is the **visualization layer** on top — dashboards and alerting UI that can query Prometheus (and many other data sources). Together they're the self-hosted alternative to [CloudWatch](../aws/services/cloudwatch.md), and the usual pairing for app/[Kafka](../apache-kafka/apache-kafka.md) metrics on [Kubernetes](../kubernetes/kubernetes.md).

---

## Resources

- **Deep Dives**
	- [Prometheus: Querying Basics (PromQL)](https://prometheus.io/docs/prometheus/latest/querying/basics/)
	- [Grafana: Getting Started](https://grafana.com/docs/grafana/latest/getting-started/)
	- [kube-prometheus-stack Helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

- **Docs & References**
	- [Prometheus Official Docs](https://prometheus.io/docs/introduction/overview/)
	- [Grafana Official Docs](https://grafana.com/docs/)
	- [Alertmanager Docs](https://prometheus.io/docs/alerting/latest/alertmanager/)

---

## Core Concepts

- **Pull-based scraping** — Prometheus doesn't receive pushed metrics; it **scrapes** an HTTP endpoint (usually `/metrics`) on each target at a configured interval. Apps and exporters expose metrics in Prometheus's plain-text format; Prometheus pulls and stores them.
- **Exporters** — small processes that translate a system's native stats into Prometheus's `/metrics` format when the target can't expose it natively (e.g. `kafka-exporter`, `node_exporter` for host metrics, `postgres_exporter`).
- **Time series & labels** — every metric is a name plus a set of key/value **labels** (e.g. `http_requests_total{method="GET", status="200"}`). Labels are what you filter/group by in queries — high-cardinality labels (raw user IDs, request IDs) blow up storage and query cost.
- **Metric types** — `counter` (monotonically increasing, e.g. total requests), `gauge` (can go up or down, e.g. active connections), `histogram`/`summary` (distributions, e.g. request latency buckets).
- **PromQL** — Prometheus's query language for selecting and aggregating time series, e.g. `rate(http_requests_total[5m])` for requests/sec over a 5-minute window.
- **Recording rules** — precomputed PromQL queries saved as new time series on a schedule. Used to turn an expensive or noisy raw query into a cheap, stable metric — e.g. the "% of workers active" custom HPA metric in [Kubernetes — Scaling](../kubernetes/kubernetes.md#scaling) is typically a recording rule, not a raw gauge.
- **Alertmanager** — a separate component that receives firing alerts from Prometheus (rule breached for N minutes) and handles routing, grouping, silencing, and dedup before paging Slack/PagerDuty/email. Prometheus decides *what* fired; Alertmanager decides *who gets told and how*.
- **Grafana data sources** — Grafana itself stores no metrics; it queries a configured data source (Prometheus, CloudWatch, Loki, Postgres, ...) per panel. One Grafana instance can front multiple backends.
- **Grafana dashboards & alerts** — dashboards are JSON documents (panels + PromQL queries), usually version-controlled and provisioned rather than clicked together by hand. Grafana can also fire alerts directly on a panel's query, as an alternative to Alertmanager.

---

## Prometheus vs CloudWatch

| | CloudWatch | Prometheus + Grafana |
| --- | --- | --- |
| Model | Push (services publish metrics to AWS) | Pull (Prometheus scrapes targets) |
| Setup | Zero-config for AWS-native resources | You deploy/operate it (or pay for a managed variant) |
| Custom app/Kafka metrics | Possible via SDK or [EMF](../aws/services/cloudwatch.md), more manual | First-class — just expose `/metrics` |
| Query language | Metric Math / Logs Insights | PromQL |
| Cost model | Pay per metric/alarm/dashboard | Self-hosted: infra cost. Managed (AMP): pay per sample |

CloudWatch is the zero-setup default on AWS. Prometheus + Grafana earn their keep once you need richer custom application metrics, Kafka consumer lag as a first-class scaling signal, or a single dashboarding tool across mixed infrastructure. See the trade-off discussion in [Observability — deploy-to-ecs-node-modulith case study](../../topics/infrastructure/case-studies/deploy-to-ecs-node-modulith.md#observability).

---

## Running on Kubernetes

The common path is the `kube-prometheus-stack` Helm chart, which bundles Prometheus, Alertmanager, Grafana, and a set of default dashboards/exporters into one release.

Prometheus needs to discover *what* to scrape. On K8s it does this via **service discovery** against the Kubernetes API — annotate a pod or Service and Prometheus picks it up automatically instead of a static target list:

```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "9090"
  prometheus.io/path: "/metrics"
```

Custom app metrics scraped this way are what feed the HPA in [Kubernetes — Scaling signals](../kubernetes/kubernetes.md#scaling): a `targetValue` in the HPA config is compared against a live PromQL query, same mechanism whether the metric is a custom app gauge or `kafka_consumergroup_lag_sum`.

### On Fargate / serverless containers

Pull-based scraping assumes stable, reachable targets. Fargate tasks have no static IP and no persistent local disk, which breaks the two things self-hosted Prometheus wants:

- **Discovery** — no static task IPs to add to a scrape config; needs ECS service discovery or a sidecar registering targets dynamically.
- **Storage** — Prometheus's local TSDB needs a persistent volume; Fargate has none by default, so it needs an EFS mount.

**Amazon Managed Prometheus (AMP)** + the **ADOT collector** sidesteps both — the collector runs as a sidecar/daemon and remote-writes to AMP instead of Prometheus scraping and storing locally, at the cost of a separate managed-service bill.

---

## Quick reference

```promql
# Requests/sec over the last 5 minutes
rate(http_requests_total[5m])

# p95 latency from a histogram
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Kafka consumer lag by topic (what drives lag-based HPA scaling)
kafka_consumergroup_lag_sum{topic="my-topic"}
```

```bash
# Install kube-prometheus-stack into a cluster
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

---
