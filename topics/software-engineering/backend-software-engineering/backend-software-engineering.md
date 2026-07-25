# Backend Software Engineering

#topic 

---
## Resources

- **Deep Dives**
	- [Authentication](assets/authentication.md)
	- [Database Normalization](assets/database-normalization.md)
	- [Scaling Databases](assets/scaling-databases.md)

---
## Internet & HTTP

The Internet is a global network that uses TCP/IP protocols for communication.

**HTTP:** A request-response protocol that defines how messages are formatted and transmitted, and how web servers and browsers should respond.

- **Methods:** `GET` (retrieve), `POST` (submit), `PUT` (replace), `PATCH` (modify), `DELETE` (remove)
- **Status Codes:** `1xx` (Informational), `2xx` (Success), `3xx` (Redirection), `4xx` (Client Error), `5xx` (Server Error)

---
## Hosting & DNS

**Hosting** provides server resources to store and deliver applications over the internet:

- **Shared Hosting:** Multiple websites on one server
- **Dedicated Hosting:** One server per user
- **Cloud Hosting:** Scalable, distributed resources
- **VPS (Virtual Private Server):** Isolated environments on a shared server

**DNS (Domain Name System):** Translates domain names into IP addresses.

---
## Databases

### Relational Databases

Organize data in structured tables with schemas (defined columns and data types), using **Primary keys and Constraints** for integrity, and **Foreign Keys** to define relationships.

#### Transactions

A **transaction** is a group of operations that execute as a single unit. Follows the **ACID** model.

#### ACID Model

| Property        | Description                                    |
| --------------- | ---------------------------------------------- |
| **Atomicity**   | The transaction is all-or-nothing.             |
| **Consistency** | Database remains valid before and after.       |
| **Isolation**   | Transactions don't interfere with each other.  |
| **Durability**  | Committed changes persist even after failures. |
#### Database Normalization

Improves integrity and reduces redundancy:

- **1NF:** Atomic values, unique rows    
- **2NF:** No partial dependencies
- **3NF:** No transitive dependencies
- **4NF+:** Advanced refinements

#### Data Access & Performance

- **Locking** – Prevents concurrent modifications to the same data.
- **Indexes** – Indexes improve lookup speed by avoiding full-table scans, but slow down writes because indexes must also be updated.
- **Stored Procedures** – Reusable SQL blocks with optional inputs/outputs.

### Scaling Databases

- **Vertical Scaling** - Adding more resources (CPU, RAM, etc.) to a single server to handle increased load. 
- **Horizontal Scaling** - Adding more servers to handle the load. This can be done through: 
	- **Sharding**: A method where a database is split into smaller, more manageable parts (shards), which are distributed across multiple servers, enhancing search efficiency and overall performance. 
	- **Read Replicas**: A form of horizontal scaling where copies of a primary database are made. These replicas handle read traffic, distributing the load, and increasing the system’s capacity to handle read requests. 
- **Database Federation** - Treating multiple databases (possibly with different engines too) as a single database.
- **Database Partitioning** - Divides data within a single database.

### CAP Theorem

> In distributed systems, you can only guarantee **two** of the three:

| Property                | Meaning                                             |
| ----------------------- | --------------------------------------------------- |
| **Consistency**         | All nodes see the same data.                        |
| **Availability**        | Every request gets a (non-error) response.          |
| **Partition Tolerance** | System continues to work during network partitions. |

### NoSQL Databases

NoSQL databases are a category of database management systems designed for handling unstructured, semi-structured, or rapidly changing data.

- **Features** 
	- **Flexible schemas** (good for fast-evolving applications)    
	- **Horizontal scalability** (better for large-scale systems)
	- Optimized for **read/write performance** and **availability**
	- Often support **eventual consistency** over strong consistency
- **Types of NoSQL databases**
	- **Key-Value stores** - A key-value store generally allows for `O(1)` reads and writes and is often backed by memory or SSD; e.g: *Redis*, *DynamoDB*
	- **Document databases** - Instead of storing data in fixed rows and columns, the data is stored in documents. e.g: *MongoDB*, *CouchDB*
	- **Wide-column**: each row can have a different set of columns. e.g *Apache Cassandra*
	- **Graph**: used for data with complex relationships - like Social Media. e.g *Neo4j*, *ArangoDB*

#### BASE Model (for NoSQL/Distributed DBs)

| Property                 | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **Basically Available**  | System always responds (even if data is stale).              |
| **Soft State**           | State can change over time, even without new input.          |
| **Eventual Consistency** | System becomes consistent *eventually*, but not immediately. |

### How to choose whether to use SQL or NoSQL database:

- **SQL database**
	- The application needs **reliable transactions** (e.g. banking, e-commerce checkout)
	- Relationships between entities are complex (foreign keys, joins)
	- There's a need for strict data validation and integrity
- **NoSQL database**
	- Schema is dynamic or **changes frequently**
	- **Speed and availability are prioritized over consistency**
	- Relationships between data are simple or irrelevant
	- You need to handle **big data / high-throughput workloads**
	- You need to scale horizontally (which is native to MongoDB for example)

### ORM (Object-Relational Mapping)

Maps DB tables to classes (e.g., Sequelize, Hibernate, TypeORM) for easier data manipulation.

### Migrations

Version-controlled schema changes using tools like TypeORM, Flyway, or Liquibase.

---
## APIs

APIs allow software components to communicate:

- **REST:** Stateless, resource-based APIs. 
- **GraphQL:** Flexible querying with a single endpoint. 
- **gRPC:** Efficient, binary-based communication. 
- **OpenAPI (Swagger):** Standardized API documentation.

---
## Authentication & Authorization

**Authentication**: Proves identity (e.g., JWT, sessions)

**Authorization**: Grants access to resources (e.g., OAuth)

- **JWT** - Open standard for securely transmitting information between parties as a JSON object. A common way to use JWTs is OAuth Bearer Tokens. 
- **OAuth** - Open standard for authorization that allows third-party applications to access a user’s resources without exposing their credentials. 
- **Basic Authentication** - Simple HTTP authentication scheme built into the HTTP protocol. It works by sending a user’s credentials (username and password) encoded in base64 format within the HTTP header. 
- **Token-based authentication** - Token-based authentication is a protocol which allows users to verify their identity, and in return receive a unique access token. JWT is the most popular implementation of token-based authentication nowadays. 
- **Cookies / Session-based Authentication** - Server-managed authentication. When a user logs in, the server creates a session and sends a unique identifier (session ID) to the client as a cookie. This cookie is then sent with every subsequent request, allowing the server to identify and authenticate the user.

---
## Distributed Systems

[System Design Topic - Distributed Systems](../system-design/system-design.md#Distributed%20Systems)

---
## Architectural Styles 

[Software Design and Architecture Topic - Architectural Styles](../software-design-and-architecture/software-design-and-architecture.md#Architectural%20Styles)

---
## Object-Oriented Programming

[Software Design and Architecture Topic - OOP](../software-design-and-architecture/software-design-and-architecture.md#Object-Oriented%20Programming)

---
## Web Security

Web security refers to protective measures taken by developers to safeguard web applications from threats that could impact the business.

### Hashing, Encryption, and Encoding

- **Hashing** — irreversible (one-way) function producing a fixed-length output called a hash, uniquely representing the input. E.g., SHA, MD5  
- **Encryption** — reversible (two-way) function transforming input into ciphertext and back. E.g., AES, RSA  
- **Encoding** — reversible transformation of data into a specified format. E.g., base64.

### Public-key Cryptography

Also known as asymmetric cryptography. Uses a pair of keys — private and public.  

Messages are signed using the **private key**, and authenticity can be verified with the **public key**. Without the private key, it is computationally infeasible to derive it from the public key.

### Secure Communication Protocols

- **TLS/SSL (HTTPS)**: Protocols securing data transmission over networks via encryption.  
- **SSH**: Secure Shell protocol for encrypted remote login and command execution.  
- **OAuth / OpenID Connect**: Protocols for delegated authorization and authentication in modern web apps.

### Common Web Vulnerabilities

- **SQL Injection**: Injecting malicious SQL to manipulate databases via input.  
- **Cross-Site Scripting (XSS)**: Injecting malicious scripts into trusted websites, executing in other users’ browsers.  
- **Cross-Site Request Forgery (CSRF)**: Forcing authenticated users to perform unintended actions.  
- **Insecure Direct Object References (IDOR)**: Accessing unauthorized resources by manipulating IDs.  
- **Security Misconfiguration**: Poor configuration of servers, frameworks, or databases leading to vulnerabilities.

---
## Caching

[System Design Topic - Caching](../system-design/system-design.md#Caching)

---
## CI/CD & Testing

Automates build and deployment:

- **CI/CD:** Automated testing, builds, deployment — see [Deployment & Release Engineering](../../infrastructure/deployment-and-release-engineering/deployment-and-release-engineering.md)
- **Tests:** Unit, Integration, Functional

---
## Containers & Virtualization

- **Virtualization:** Creates separate virtual machines (VMs), each with its own operating system, running on a hypervisor. This provides strong isolation but consumes more resources. 
- **Containers (Docker):** Uses a shared operating system kernel to create isolated environments (containers) for applications. Containers are lighter, start faster, and use fewer resources than VMs. They’re ideal for microservices architectures and rapid deployment.
- **Orchestration (Kubernetes):** Manages containerized applications.

---
## Web Servers

Serve and process HTTP content:

- **Static web servers** - A static web server, or stack, consists of a computer (hardware) with an HTTP server (software). We call it "static" because the server sends its hosted files as-is to your browser. 
- **Dynamic web servers** - A dynamic web server consists of a static web server plus extra software, most commonly an application server and a database. We call it "dynamic" because the application server updates the hosted files before sending content to your browser via the HTTP server.

**Popular Servers:** Nginx, Apache, IIS

---
## Real-Time Data

Deliver low-latency data to clients:

- **WebSockets:** Bi-directional, persistent
- **SSE:** Server-push over HTTP
- **Long Polling:** Hold open until data
- **Short Polling:** Periodic checks

---
## Observability

Ability to infer internal system state from outputs:

- **Metrics:** Numeric time-series
- **Logs:** Text-based events
- **Traces:** End-to-end request path

**Tools:** [Prometheus & Grafana](../../../technologies/prometheus-grafana/prometheus-grafana.md), ELK Stack, OpenTelemetry

---