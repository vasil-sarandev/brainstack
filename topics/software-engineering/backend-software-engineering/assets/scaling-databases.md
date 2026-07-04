# Scaling Databases

#deep-dive

Scaling databases is the process of adapting them to handle more data and users efficiently.

It’s achieved by either upgrading existing hardware (vertical scaling) or adding more servers (horizontal scaling).

---
## Database Indexes

Database indexes are data structures that improve the speed of data retrieval operations in a database management system. They work similarly to book indexes, providing a quick way to look up information based on specific columns or sets of columns.

Indexes create a separate structure that holds a reference to the actual data, allowing the database engine to find information without scanning the entire table.

While indexes significantly enhance query performance, especially for large datasets, they come with trade-offs. They increase storage space requirements and can slow down write operations as the index must be updated with each data modification.

---
## Strategies for Scaling Databases

### Caching

Caching is a technique used to store frequently accessed data in memory, making subsequent access to that data faster. It’s a common strategy used to improve the performance of a database system by reducing the need to access the underlying database.

Use cases:

- **Read-Heavy Workloads**
- **Temporal Data Locality**: If the same set of data is frequently accessed within short periods of time, caching that data can improve performance.
- **Expensive Computations**: If the application performs complex queries or computations that take a long time to complete, we can cache the results to speed up subsequent requests.
- **Rate Limiting**: If you need to limit the number of requests to your database to prevent overloading it, caching can help by reducing the number of direct database accesses.

### Vertical Scaling

Vertical scaling is often called “scaling up”. It’s the process of upgrading the hardware of your existing server. i.e., more CPU, RAM, Storage etc. Vertical scaling is a straightforward and quick approach, especially using cloud-based database services.

Use cases:

- **Monolithic Applications**: Applications that are not designed to be distributed across multiple servers can benefit from vertical scaling.
- **Database Servers**: Databases often benefit from vertical scaling because they require fast access to data, which can be more efficiently achieved on a single powerful server.
- **Temporary Scaling**: Vertical scaling can be a good option when you need to quickly handle a temporary spike in traffic or load.

### Read Replicas (Horizontal Scaling)

Read replicas are a form of horizontal scaling where you create one or more copies of your primary database. These replicas are kept in sync with the primary database and can handle read traffic, thereby distributing the load, and increasing the system’s capacity to handle read requests.

Async replication keeps all the replicas in sync with the main database but there might be a small delay. Hence immediate read after write might result in stale data.

Use cases:

- **Read-Heavy Workloads**: If your application performs many more read operations than write operations, you can use read replicas to distribute the read load across multiple databases.
- **Data Reporting**: If you have reporting tasks or analytics that require heavy database reads, you can direct these tasks to a read replica. This way, your reporting tasks won’t impact the performance of your primary database that handles your application’s operational workload.
- **Geographic Distribution**: If you have users in different geographic locations, you can improve your application’s performance by placing read replicas in the regions closer to your users.
- **Database Backups**: You can use a read replica to take backups of your data. This way, your backup operations won’t impact the performance of your primary database.
- **Failover Support**: In case your primary database fails, a read replica can be promoted to become the new primary database, providing high availability.

### Sharding (Horizontal Scaling)

Relational databases slow down as the volume of records in a table increases. Searching for an item in such a large table, especially without indexing, becomes progressively slower as the data size expands. This often leads to performance bottlenecks as the database grows.

Implementing sharding is an effective strategy to overcome these challenges, as it divides the large datasets into manageable segments, enhancing search efficiency and overall performance.

![Shards example](shards-example.png)

Use cases:

- **Large Databases**: Sharding can help manage large databases that are not able to be handled efficiently by a single server.
- **High Traffic**: For applications with high traffic, sharding can help by distributing the load across multiple servers.
- **Geographical Distribution**: If your application has users in different geographical locations, you can improve performance by placing shards in the regions closer to your users.

---