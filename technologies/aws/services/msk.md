# AWS MSK (Managed Streaming for Apache Kafka)

#technology #aws #kafka

**MSK** is AWS's managed Kafka — brokers, storage, and patching handled by AWS; you still own topics, partitions, client auth, and connectivity. Two flavors: **Provisioned** (you size brokers/storage yourself) and **Serverless** (no capacity planning, but IAM-auth-only).

Part of: [Amazon Web Services](../aws.md). Lives inside a [VPC](vpc.md); consumed by [ECS](ecs.md) tasks or any Kafka client. See [Apache Kafka](../../apache-kafka/apache-kafka.md) for the protocol/concepts themselves — this page is the AWS-specific operational layer on top.

---

## Resources

- [Amazon MSK Developer Guide](https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html)
- [Client authentication](https://docs.aws.amazon.com/msk/latest/developerguide/kafka_client_auth.html)
- [Encryption in transit](https://docs.aws.amazon.com/msk/latest/developerguide/msk-encryption.html)

---

## Core Concepts

- **Cluster**
  A set of **brokers** (minimum 2, spread across AZs) plus **Zookeeper/KRaft** metadata management, handled by AWS. Provisioned clusters need explicit broker type/count/storage; Serverless auto-scales partitions/throughput but bills per-partition-hour instead.

- **Broker instance types**
  `kafka.t3.small` exists but is explicitly documented by AWS as dev/test only — burstable CPU credits throttle under real load. `kafka.m5.large` is the practical floor for anything resembling production. Minimum 2 brokers means you're always paying for at least 2× whatever instance you pick.

- **Bootstrap servers**
  The comma-separated broker connection strings a client actually connects to — different lists per auth/encryption mode (plaintext `:9092`, TLS `:9094`, SASL/IAM `:9098`, SASL/SCRAM `:9096`). Find them via **cluster → View client information**.

- **Client authentication vs. encryption in transit — two separate axes**
  - **Client authentication** (IAM / SASL-SCRAM / TLS mutual auth / Unauthenticated) — editable after cluster creation via **Edit/Delete → "Edit security settings."**
  - **Encryption in transit** (whether a plaintext listener exists at all, vs. TLS-only) — chosen once at cluster creation and **not editable afterward**. If you need plaintext and didn't enable it at creation, there's no console path to add it later short of recreating the cluster.
  - "Quick create" mode defaults to **IAM authentication only**, no plaintext or TLS-unauthenticated listener — a common first-cluster surprise if you expected a plain Kafka connection to just work.

- **Security group is fixed at creation**
  Unlike most AWS networking, the VPC security group(s) attached to an MSK cluster **cannot be swapped** after creation — the console's networking "Edit" only covers public-access and multi-VPC-connectivity toggles. Workaround: add the needed inbound rule to whichever SG is already attached, rather than trying to reattach a purpose-named one.

- **No topic auto-creation**
  `auto.create.topics.enable` is `false` by default. Nothing creates topics for you — something with network access *inside the VPC* has to run `kafka-topics.sh --create` (or an admin-client equivalent) against the real bootstrap servers. A throwaway EC2 instance (reached via SSM Session Manager, no SSH/bastion SG needed) or a one-off Fargate task using the same image/network as your real services both work; the latter needs no new infrastructure if you're already running on ECS.

---

## IAM auth vs. plaintext/TLS — what it costs you

| | Plaintext / TLS-unauthenticated | SASL/IAM |
| --- | --- | --- |
| **Client code** | Works with any Kafka client library unmodified | Needs an IAM-SASL-signer library (e.g. `aws-msk-iam-sasl-signer-js` for kafkajs) wired into the SASL config |
| **App permissions** | None needed | Task/app needs its own IAM role granting `kafka-cluster:Connect` / `DescribeTopic` / `ReadData` / `WriteData` |
| **Setup** | Simplest if encryption-in-transit allows it | More AWS-native, but real code + IAM work |

If you're prototyping and don't want a client-library change, plaintext or TLS-unauthenticated is the path of least resistance — just remember encryption-in-transit has to allow it from cluster creation, not something to bolt on later.

---

## Cost reality

No single-broker option — minimum 2 brokers × whatever instance type, running continuously, is the baseline. Ballpark for the cheapest legitimate option (`kafka.t3.small` × 2) left running 24/7: roughly $60-90/month, mostly compute. A spin-up/test/teardown session costs a few dollars, not the monthly figure — the actual risk is forgetting to delete the cluster afterward. An AWS Budget alert on MSK spend is cheap insurance against exactly that.

---
