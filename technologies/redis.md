# Redis

#technology

Redis is an open-source, **in-memory data store** used as a database, cache, and message broker. It offers sub-millisecond latency, making it ideal for performance-critical use cases.

---

## Resources

- **Deep Dives / Hands-on**

  - [Redis Commands Cheatsheet - Redis.io](https://redis.io/learn/howtos/quick-start/cheat-sheet)
  - [API Gateway Caching with Redis](https://redis.io/learn/howtos/solutions/microservices/api-gateway-caching)
  - [Interservice Communication with Redis](https://redis.io/learn/howtos/solutions/microservices/interservice-communication)
  - [Redis for Query Caching](https://redis.io/learn/howtos/solutions/microservices/caching)
  - [Microservices Architecture with Redis](https://redis.io/learn/howtos/solutions/microservices/common-data/microservices-arch)
  - [Real-time Chat App with Redis & AWS](https://redis.io/learn/create/aws/chatapp)

- **Docs & References**
  - [Redis Official Docs](https://redis.io/docs/latest/)

---

## Core Concepts

- **In-Memory**: Data is stored in RAM → extremely fast reads/writes.
- **Data Structures**: Supports strings, lists, sets, hashes, sorted sets, streams, hyperloglogs, and bitmaps.
- **Persistence Options**:
  - **RDB (Redis Database Backup)** - RDB creates point-in-time snapshots of the dataset at specified intervals, offering efficient storage with minimal performance impact, making it suitable for periodic backups but with potential data loss between snapshots.
  - **AOF (Append-Only File)** - AOF logs every write operation to disk, providing higher data durability by allowing finer-grained recovery, though it can be more resource-intensive.
- **Typical use cases**:
  - **Caching** (query results, API responses, sessions)
  - **Real-time messaging** (via Pub/Sub or Streams)
  - **Rate limiting** (token bucket, sliding window)
  - **Queue systems** (using lists or streams)
  - **Leaderboard / Ranking** (with sorted sets)
  - **Distributed locks** (e.g., with Redlock algorithm)

---
