# System Design

#topic

In software engineering, system design is a phase in the software development process that focuses on the high-level design of a software system, including the architecture and components, and how they interact with each other.

---
## Resources

- **Deep Dives**
	- [How to do a System Design of a Software Application](assets/how-to-do-a-system-design.md)

- **Case Studies** 
	- [TicketMaster System Design](assets/case-studies/system-design-ticketmaster.md)
	- [Bitly System Design](assets/case-studies/system-design-bitly.md)
	- [Twitter System Design](assets/case-studies/system-design-twitter.md)
	- [Instagram System Design](assets/case-studies/system-design-instagram.md)

- **Other Resources**
	- [System Design roadmap - roadmap.sh](https://roadmap.sh/system-design)
	- https://www.hiredintech.com/system-design/ (System Design Interview guide, **this one is pretty great.**)
	- https://www.youtube.com/@hello_interview (System Design Youtube Channel)
	- https://github.com/donnemartin/system-design-primer (System Design Primer)

---
## General Terminology

### Performance vs Scalability

- If you have a **performance** problem, your system is slow for a single user.
- If you have a **scalability** problem, your system is fast for a single user but slow under heavy load.

### Latency vs Throughput

- **Latency** refers to the amount of time it takes for a system to respond to a request.
- **Throughput** refers to the number of requests that a system can handle at the same time.

### Availability vs Consistency

- **Availability** refers to the ability of a system to provide its services to clients even in the presence of failures.
- **Consistency** refers to the property that all clients see the same data at the same time.

### Resiliency

Resiliency is the ability of a system to gracefully handle and recover from failures, both inadvertent and malicious.

---
## Distributed Systems

Distributed systems consist of components spread across multiple machines, working together to achieve a common goal.

### Horizontal vs Vertical Scaling

- **Horizontal Scaling**: Adding more machines/servers to the system.  
  ↳ Improves **resilience** and allows **independent failure**.

- **Vertical Scaling**: Adding more resources (RAM, CPU, etc.) to a single server.  
  ↳ Simpler but has **hardware limits** and **a single point of failure**.

### CAP Theorem

According to the CAP theorem, in a distributed system, you can only support two of the following guarantees:

- **Consistency** - Every read receives the most recent write or an error
- **Availability** - Every request receives a response, without guarantee that it contains the most recent version of the information
- **Partition Tolerance** - The system continues to operate despite arbitrary partitioning due to network failures

### Consistency Models

Consistency Models refer to the ways in which data is stored and managed in a distributed system, and how that data is made available to users and applications. There are three main types of consistency patterns:

 - **Strong Consistency** - After an update is made to the data, it will be immediately visible to any subsequent read operations. The data is replicated in a synchronous manner, ensuring that all copies of the data are updated at the same time.
 - **Eventual (or Weak) Consistency** - After an update is made to the data, it will be eventually visible to any subsequent read operations. The data is replicated in an asynchronous manner, ensuring that all copies of the data are eventually updated.

### Distributed Transactions

A single operation that spans multiple microservices. They are typically achieved with:

- **2PC (2-Phase Comitt)**: A synchronous blocking protocol for strict consistency. Coordinator sends a prepare request and only after all participants acknowledge, is it committed.
- **SAGA** - An asynchronous sequence of local transactions for high throughput. Each step is committed locally and if a step fails, all previous steps trigger an "undo transaction" (not a rollback, since the transaction is complete already).

### Fault Tolerance

- **Leader Election**: Ensures coordination in distributed environments.
- **Circuit Breaker Pattern**: Prevents cascading failures in microservices.
- **Retries & Exponential Backoff**: Gracefully handles transient failures.

### Messaging

Message Queues and Brokers are intermediaries that enable **asynchronous communication** between distributed system components, decoupling producers (senders) from consumers (receivers) to improve scalability, reliability, and flexibility.

- **Message Queues**  
    Store messages temporarily until consumers retrieve them, often following **FIFO** (First In, First Out) order. They provide reliable delivery with features like acknowledgment and retry. Commonly used for task queues, load leveling, and buffering.
- **Message Brokers**  
    More advanced middleware that routes, transforms, filters, and manages messages across multiple producers and consumers. They support complex messaging patterns like **pub/sub**, topic routing, and message filtering.

**Examples:** RabbitMQ, Apache Kafka, Amazon SQS, Azure Service Bus.

---
## Databases

[Backend Software Engineering Topic - Databases](../backend-software-engineering/backend-software-engineering.md#Databases)

---

## Caching

Stores frequently accessed data to reduce latency and database load.

### Types

- **Client-side:** Browser cache, service workers  
- **Server-side:** In-memory stores (e.g. **Redis**, **Memcached**)  
- **CDNs (Content Delivery Networks):** Cache static assets closer to users at the edge  

### Strategies

- **Cache-aside (Lazy Loading):**  App checks cache first. On miss, fetches from DB, updates cache.  E.g. Redis, Memcached.
- **Write-through:**  Writes go to cache **and** database at the same time.  
- **Write-behind**: Writes go to cache first, then asynchronously to DB. 
- **Refresh-ahead**:  Cache refreshes entries **before** they expire if accessed recently.  

### Eviction Policies

When the cache is full, older items are evicted based on policy:

- **LRU (Least Recently Used):**  Removes items that haven’t been used for the longest time.  
- **LFU (Least Frequently Used):**  Removes items used the least often.  
- **MRU (Most Recently Used):** Removed the item that was most recently used.
- **TTL (Time To Live):**  Items expire automatically after a set time. 

---
## Availability Patterns

Availability is measured as a percentage of uptime, and defines the proportion of time that a system is functional and working. Availability is affected by system errors, infrastructure problems, malicious attacks, and system load

### Replication

Replication is an availability pattern that involves having multiple copies of the same data stored in different locations. In the event of a failure, the data can be retrieved from a different location. There are 2 main types of replication:

- **Master-Slave Replication** - In this type of replication, one server is designated as the "master" and handles all write operations, while multiple "slave" servers handle read operations. If the master fails, one of the slaves can be promoted to take its place.
- **Master-Master Replication** - In this type of replication, multiple servers are configured as "masters," and each one can accept read and write operations. This allows for high availability and allows any of the servers to take over if one of them fails. However, this type of replication can lead to conflicts if multiple servers update the same data at the same time, so some conflict resolution mechanism is needed to handle this.

### Fail-Over

Failover is an availability pattern that is used to ensure that a system can continue to function in the event of a failure. It involves having a backup component or system that can take over in the event of a failure.

- **Active-Passive Fail-Over** - Heartbeats are sent between the active and the passive server on standby. If the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service.
- **Active-Active Fail-Over** - In active-active, both servers are managing traffic, spreading the load between them.

---
## Background Jobs

Background jobs in system design refer to tasks that are executed in the background, independently of the main execution flow of the system. These tasks are typically initiated by the system itself, rather than by a user or another external agent. 

They can be *Event* or *Schedule Driven*. Use cases include: 

- Performing maintenance tasks: such as cleaning up old data or generating reports.
- Processing large volumes of data: such as data import, data export, or data transformation.
- Sending notifications or messages: sending email notifications or push notifications to users.
- Performing long-running computations: such as machine learning or data analysis.

---
## Content Delivery Networks

A content delivery network (CDN) is a globally distributed network of proxy servers, serving content from locations closer to the user.

- **Push CDNs** - Push CDNs receive new content whenever changes occur on your server. You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN. Push CDNs are good for small websites or ones where content doesn't update often.
- **Pull CDNs** - Pull CDNs grab new content from your server when the first user requests the content. You leave the content on your server and rewrite URLs to point to the CDN. Pull CDNs work well with traffic-heavy websites.

---
## Load Balancer, Reverse Proxy & API Gateway

### Load Balancer

Load balancers distribute incoming client requests to computing resources such as application servers and databases. In each case, the load balancer returns the response from the computing resource to the appropriate client. Load balancers are effective at:

- Preventing requests from going to unhealthy servers
- Preventing overloading resources
- Helping to eliminate a single point of failure

### Reverse Proxy

A reverse proxy server sits in front of web servers and forwards client (e.g. web browser) requests to those web servers. The requested resources are then returned to the client, appearing as if they originated from the proxy server itself.

Some of the benefits a Reverse Proxy provides are:

- **Increased security** -no information about your backend servers is visible outside your internal network, so malicious clients cannot access them directly to exploit any vulnerabilities. 
- **Increased scalability and flexibility** - because clients see only the reverse proxy’s IP address, you are free to change the configuration of your backend infrastructure. 
- **Web acceleration** - through *Intelligent Compression*, *Caching*, or *SSL acceleration*.

### API Gateway

An **API Gateway** is a specialized type of _reverse proxy_ designed to act as a single entry point for client requests in a microservices architecture. It routes incoming API calls to the appropriate backend services and often handles cross-cutting concerns that would otherwise be duplicated across services.

Gateways are most often implemented with cloud native solutions like AWS API Gateway and their common responsibilities include: 

- **Authentication and Authorization** – validating tokens and controlling access to services.    
- **Rate Limiting and Throttling** – protecting services from abuse by limiting request rates.
- **Request and Response Transformation** – modifying headers, payloads, or protocols.
- **Logging and Monitoring** – capturing metrics and request logs for observability.
- **Caching** – storing responses to reduce load on backend services.



### Load Balancers vs Reverse Proxies

While they serve similar purposes, Load Balancers are most commonly deployed when a site needs multiple servers because the volume of requests is too much for a single server to handle efficiently. Reverse Proxies often make sense be deployed even with a single server so they can serve as the application's "public face".

---
## Service Discovery and Service Mesh

### Service Discovery

_Service discovery_ helps you discover, track, and monitor the health of services within a network. Service discovery registers and maintains a record of all your services in a _service catalog_. This service catalog acts as a single source of truth that allows your services to query and communicate with each other.

A service consumer communicates with the "Web" service via a unique DNS entry provided by the service catalog (such as Consul, Etcd, Apache Zookeper).

### Service Mesh

*Service Discovery* is great for small systems, but large complex systems often use a dedicated infrastructure layer that facilitates communication and discovery between services called a Service Mesh.

The service mesh serves as a powerful design pattern that abstracts the underlying network infrastructure, providing a standardized solution by deploying sidecar proxies alongside your services. These proxies, often leveraging technologies like the Envoy proxy, handle critical networking tasks, security enforcement, and observability.

---
## Idempotent Operations

Idempotent operations are operations that can be applied multiple times without changing the result beyond the initial application.

Many queueing systems guarantee _at least once_ message delivery or processing. Designing the operations that a task queue executes to be idempotent allows one to use a queueing system that has accepted this design trade-off.

---
## Communication

Network protocols are a key part of systems today, as no system can exist in isolation - they all need to communicate with each other.

- **HTTP**
	HTTP is a method for encoding and transporting data between a client and a server. It is a request/response protocol: clients issue requests and servers issue responses with relevant content and completion status info about the request.
- **TCP**
	TCP is a connection-oriented protocol over an IP network. Connection is established and terminated using a handshake. All packets sent are guaranteed to reach the destination in the original order and without corruption.
- **UDP**
	UDP is connectionless. Datagrams (analogous to packets) are guaranteed only at the datagram level. Datagrams might reach their destination out of order or not at all. UDP does not support congestion control. Without the guarantees that TCP support, UDP is generally more efficient and often used in scenarios where **low latency** is critical, like video streaming, gaming, or DNS queries.
- **RPC (Remote Procedure Call)**
	RPC is a Request-Response Protocol. In an RPC, a client causes a procedure to execute on a different address space, usually a remote server. The procedure is coded as if it were a local procedure call, abstracting away the details of how to communicate with the server from the client program.
- **gPRC** 
	gRPC is a high-performance, open-source RPC framework developed by Google. It uses HTTP/2 as transport and Protocol Buffers for serialization, enabling efficient and strongly-typed communication. gRPC supports unary and streaming calls (both client and server streaming), making it very suitable for microservices communication and real-time applications.
- **REST**
	REST is an architectural style enforcing a client/server model where the client acts on a set of resources managed by the server. The server provides a representation of resources and actions that can either manipulate or get a new representation of resources. All communication must be stateless and cacheable. The HTTP methods are typically mapped to CRUD operations and JSON or XML is used for payloads.
- **GraphQL** 
	GraphQL is a query language and runtime for building APIs. It allows clients to define the structure of the data they need and the server will return exactly that. This is in contrast to traditional REST APIs, where the server exposes a fixed set of endpoints and the client must work with the data as it is returned.
- **Message Queues / Message Brokers** 

---
## Performance Antipatterns

Performance antipatterns in system design refer to common mistakes or suboptimal practices that can lead to poor performance in a system.

- **Improper instantiation** 
	Classes that are supposed to only be instantiated once are created on demand.
- **Monolithic Persistence** 
	The use of a single, monolithic database to store all of the data for an application or system. This approach can be used for simple, small-scale systems but as the system grows and evolves it can become a bottleneck, resulting in poor scalability, limited flexibility, and increased complexity. Solve with Microservices, Sharding, and noSQL databases.
- **Noisy Neighbour** 
	A situation in which one or more components of a system are utilizing a disproportionate amount of shared resources, leading to resource contention and reduced performance for other components.
- **Synchronous I/O** 
	A synchronous I/O operation blocks the calling thread while the I/O completes. The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.
- **Extraneous Fetching** 
	Retrieving more data than is needed for a specific task or operation.
- **Busy Database**
	When a database is not properly optimized for the workload it is handling. This can lead to Performance degradation, Increased resource utilization, Deadlocks and contention.
- **Chatty I/O**
	The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness. e.dg - reading and writing individual records to a database as distinct requests.
- **Retry Storm**
	A situation in which a large number of retries are triggered in a short period of time, leading to a significant increase in traffic and resource usage.
- **No caching** 
	When a cloud application that handles many concurrent requests, repeatedly fetches the same data. This can reduce performance and scalability.

---
## Monitoring

- **Health Monitoring** 
	A system is healthy if it is running and capable of processing requests. The purpose of health monitoring is to generate a snapshot of the current health of the system so that you can verify that all components of the system are functioning as expected. 
- **Availability Monitoring** 
	Availability monitoring is concerned with tracking the availability of the system and its components to generate statistics about the uptime of the system. 
- **Performance Monitoring** 
	As the system is placed under more and more stress, performance degradation is likely. Frequently, component failure is preceded by a decrease in performance, so it's important to monitor performance.
- **Security Monitoring** 
	All commercial systems that include sensitive data must implement a security structure. The complexity of the security mechanism is usually a function of the sensitivity of the data.
- **Usage Monitoring** 
	Usage monitoring tracks how the features and components of an application are used.
- **Instrumentation** 
	Instrumentation is a critical part of the monitoring process. You can make meaningful decisions about the performance and health of a system only if you first capture the data that enables you to make these decisions. The information that you gather by using instrumentation should be sufficient to enable you to assess performance and diagnose problems. 
- **Common Tools**
	- Open Telemetry - Instrumentation
	- Prometheus + Grafana - Health, Availability & Performance Monitoring
	- DataDog / NewRelic - Health, Availability & Performance Monitoring

---
## Messaging Design Patterns

 **Sequential Convoy** 
	A pattern that allows for the execution of a series of tasks, or convoy, in a specific order. This pattern can be used to ensure that a set of dependent tasks are executed in the correct order and to handle errors or failures during the execution of the tasks.
- **Scheduler-Agent-Supervisor** 
	Coordinate a set of distributed actions as a single operation. If any of the actions fail, try to handle the failures transparently, or else undo the work that was performed, so the entire operation succeeds or fails as a whole.
- **Pub/sub (publish-subscribe) pattern** 
	Enable an application to announce events to multiple interested consumers asynchronously, without coupling the senders to the receivers. This is the opposite of Message Queues where a message is consumed only by a single subscriber. Popular Message Queues / Brokers offer implementations for pub/sub.
- **Priority Queue** 
	Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.
- **Pipes and Filters** 
	Decompose a task that performs complex processing into a series of separate elements that can be reused. This can improve performance, scalability, and reusability by allowing task elements that perform the processing to be deployed and scaled independently. E.g - CI/CD : lint, test, build, deploy, ...
- **Choreography**
	Each service listens for and reacts to events on a shared message bus, then emits new events without a central coordinator. Services coordinate indirectly through event chains, enabling loose coupling but requiring careful design to manage flow, errors, and dependencies.
- **Claim-Check pattern**
	In order to not overload our messaging service / queue, messages should be kept small. If we need to include large data, we can replace it with a claim-check token that allows another service to retrieve the large data from a data store, instead of sending it to the message queue.

---
## Design & Implementation 

- **Strangler Fig** 
	Incrementally migrate a legacy system by gradually replacing specific pieces of functionality with new applications and services.
- **Sidecar Pattern** 
	Deploy components of an application into a separate process or container to provide isolation and encapsulation. 
	The Sidecar is attached to a parent application and provides supporting features for the application. The sidecar also shares the same lifecycle as the parent application, being created and retired alongside the parent.
	E.x - Authentication. The Sidecar can intercept requests and add API keys, Bearer Tokens or it can intercept and validate incoming requests. Other examples: logs, metrics, proxying
- **Leader Election**
	Coordinate the actions performed by a collection of collaborating instances in a distributed application by electing one instance as the leader that assumes responsibility for managing the others. In essence it's a distributed Mutex that's managed by the Leader, so instances don't overwrite each other's changes.
- **CQRS - Command and Query Responsibility Segregation** 
	A pattern that separates read and update operations for a data store. Implementing CQRS in your application can maximize its performance, scalability, and security.
- **Ambassador** 
	An Ambassador Service is a specific type of *Sidecar* that acts as a proxy between our main application and the Remote Services. The Ambassador handles implementations such as Retry, Authentication, Circuit Breaking, Logs, Monitoring, Security.
- **Gateway Routing** 
	Route requests to multiple services or multiple service instances using a single endpoint.
- **Gateway Aggregation** 
	This pattern is useful when a client must make multiple calls to different backend systems to perform an operation. With Gateway Aggregation, the Gateway Makes the requests, combines their responses, and sends the aggregated result to the client.
- **External Config Store** 
	Move configuration information out of the application deployment package to a centralized location. This can provide opportunities for easier management and control of configuration data, and for sharing configuration data across applications and application instances.
- **Backend for Frontend (BFF)**
	Create separate backend services to be consumed by specific frontend applications or interfaces. This pattern is useful when you want to avoid customizing a single backend for multiple interfaces.
- **Anti-Corruption Layer** 
	Implement a facade or adapter layer between different subsystems that don't share the same semantics. This layer translates requests that one subsystem makes to the other subsystem.


--- 
