# PostgreSQL

#technology

PostgreSQL is a powerful, open-source object-relational database system known for its reliability, feature richness, standards compliance, and extensibility.

---
## Resources

- **Tutorials**
	- [PostgreSQL Tutorial - neon.com](https://neon.com/postgresql/tutorial#quick-start)
- **Docs and References**
	- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/)

---
## Core Concepts

- **Transaction**: A sequence of SQL operations performed as a single logical unit of work. Supports **ACID** properties (Atomicity, Consistency, Isolation, Durability).
- **Typical use cases**:
	- **Transactional applications** (e.g., finance, e-commerce)
	- **Data warehousing and analytics**
	- **Geospatial data processing**
	- **Custom data types and procedures**
	- **Time-series data management**

---

## PSQL

**PSQL** is the interactive command-line tool for working with PostgreSQL databases. It allows you to execute SQL queries, manage database objects, and interact with the database directly from your terminal. It supports meta-commands (starting with `\`) for database navigation and administration, making it a powerful utility for developers and DBAs.

```c
# Connect to Server
psql -h <host> -p <port> -U <username> -d <database>

# List databases
\l

# Connect to database
\c <database>

# List tables
\dt

# Describe table structure
\d <tableName>
```

---