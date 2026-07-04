# Apache Kafka

#technology

Apache Kafka is a distributed event streaming platform designed for high-throughput, fault-tolerant, and scalable real-time data pipelines and streaming applications.

---
## Resources

- **Deep Dives**
	- [Confluent.io Kafka Deep Dive Topics](https://developer.confluent.io/learn/)

- **Docs and References**
	- [Kafka Official Documentation](https://kafka.apache.org/documentation/)
	- [Confluent.io - Official Managed Kafka Reference](https://developer.confluent.io/)

---
## Core Concepts

- **Topic**: A named stream of records (like a category or feed name).
- **Producer**: An application that publishes (writes) data to topics.
- **Consumer**: An application that subscribes to topics to read data.
- **Partition**: Topics are split into partitions for parallelism and scalability.
- **Broker**: A Kafka server that stores data and serves clients.
- **Consumer Group**: A group of consumers coordinating to consume topic partitions collectively.
- **Offset**: A unique identifier for each record within a partition, used to track consumption progress.
- **Typical use cases**:
	- **Real-time data pipelines** (ingesting and moving data between systems)
	- **Stream processing** (transforming or enriching data streams)
	- **Event sourcing and messaging** (decoupling microservices)
	- **Log aggregation** and monitoring
	- **High-throughput, fault-tolerant event storage**

---

## Partitions & Consumer Scaling

Partitions are the unit of parallelism in Kafka. A topic with N partitions can be consumed by at most N consumers in parallel — each partition is assigned to exactly one consumer in a group at a time.

```
Topic (20 partitions)
  ├── Partition 0  →  Consumer A
  ├── Partition 1  →  Consumer B
  ├── Partition 2  →  Consumer C
  ...
  └── Partition 19 →  Consumer T
```

If you have fewer consumers than partitions, some consumers handle multiple partitions. If you have more consumers than partitions, the extras sit idle — no benefit beyond the partition count.

**This means partition count = max parallelism ceiling.** You set it when creating the topic and it's expensive to change later, so it's usually set conservatively high upfront.

### How this maps to Kubernetes deployments

Each consumer process runs in its own pod. When a new pod starts, it joins the consumer group and Kafka automatically rebalances — redistributing partitions across all active consumers.

```yaml
replicaCount: 1      # start with 1 pod (1 consumer)
autoscaling:
  maxPods: 20        # matches the topic's partition count
  prometheus:
    metricName: "kafka_consumergroup_lag_sum"
    prometheusLabels:
      topic: "my-topic"
      consumergroup: "my-consumer-group"
    targetValue: 1000   # scale up when this many messages are waiting
```

The scaling loop:
1. Messages pile up → lag exceeds `targetValue`
2. HPA spins up a new pod → pod joins consumer group
3. Kafka rebalances → new pod gets assigned partitions
4. Lag drops → pods scale back down

`maxPods` should always match the partition count. More pods than partitions wastes resources; fewer means you can never fully utilise the topic's parallelism.

### Partition resource cost on brokers

Partitions aren't free — each one has a real cost on the broker side, which is why you can't set partition count arbitrarily high:

- **Disk** — each partition is a physical directory with its own log segment files on the broker, even if mostly empty
- **Memory** — brokers keep an index and metadata per partition in heap
- **File handles** — each segment file is an open file descriptor; brokers can hit OS limits with thousands of partitions
- **Replication overhead** — each partition has a leader + replicas across brokers, generating constant replication traffic
- **Controller recovery time** — during a broker failure, the controller must re-elect leaders for all affected partitions; thousands of partitions means noticeably slower recovery

The trade-off:
```
more partitions  →  higher max throughput / parallelism
                 →  more broker resource usage
                 →  slower rebalances and failure recovery
```

Set partition count to a realistic `maxPods` ceiling — there's no benefit in 200 partitions if you'll never run more than 20 consumers, and you pay the broker cost for all 200 regardless.

---

## Dead Letter Queues (DLQ)

A DLQ is a separate topic where messages are routed when a consumer fails to process them after all retries are exhausted. Instead of blocking the partition (which would halt all subsequent messages) or silently dropping the message, it gets parked in the DLQ for later inspection or reprocessing.

### The problem DLQs solve

Kafka consumers commit offsets to track progress. If a message fails and the consumer doesn't commit its offset, it will re-consume the same message on the next poll — potentially forever. This is called a **poison pill**: one bad message blocks the entire partition.

```
Partition 0:  [msg1 ✓] [msg2 ✓] [msg3 💀] [msg4 waiting...] [msg5 waiting...]
                                     ↑ consumer keeps retrying, nothing else moves
```

### How it works

1. Consumer attempts to process a message
2. On failure, retries N times (with backoff)
3. After all retries exhausted, publishes the message to the DLQ topic
4. Commits the offset and moves on — the partition is unblocked
5. A separate process (or manual intervention) handles the DLQ

```
main-topic          →  consumer fails after 3 retries  →  main-topic.dlq
                                                              ↑ parked here for later
```

### DLQ topic naming convention

Typically suffixed on the original topic name:
- `loyalty-outbox-events.points.points-state` → `loyalty-outbox-events.points.points-state.dlq`

### What to do with DLQ messages

- **Inspect** — understand why they failed (schema mismatch, downstream service down, bad data)
- **Replay** — re-publish them to the original topic once the root cause is fixed
- **Discard** — if the message is genuinely invalid and can never be processed
- **Alert** — DLQ depth is a useful metric to monitor; growing DLQ = something is consistently broken

### DLQ vs retries

Retries and DLQs serve different failure modes:
- **Retries** — handle transient failures (network blip, temporary unavailability). Should be short and bounded.
- **DLQ** — handles persistent failures (bad message format, unrecoverable error). Prevents one bad message from blocking the stream indefinitely.

---

## KRaft and ZooKeeper

- **ZooKeeper**
	- **Role**: Originally, Kafka relied on **Apache ZooKeeper** to manage cluster metadata such as broker membership, topic configurations, partition assignments, and controller election.
	- **Challenges**:
	    - Operational complexity (separate ZooKeeper cluster).
	    - Latency and inconsistency issues.
	    - Difficult to scale and maintain.
- **KRaft (Kafka Raft Metadata Mode)**
	- **Introduced**: As part of Kafka’s effort to remove the dependency on ZooKeeper.
	- **Based on**: A Raft consensus protocol implementation built into Kafka.
	- **Key Features**:
	    - Kafka brokers themselves manage metadata.
	    - One broker acts as the **controller** (leader), replicating metadata logs to others.
	    - Improved consistency, scalability, and simpler architecture.
	    - Easier deployment (no separate ZooKeeper cluster).

---