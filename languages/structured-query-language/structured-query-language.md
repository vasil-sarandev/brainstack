# Structured Query Language Cheatsheet

#language

SQL or "sequel" has been around for decades and supports a many billion dollar markets.
It is supported by all major commercial database systems and is a declarative language, based on relational algebra.

There's 2 parts to the language:

- **Data Definition Language** (DDL) - CREATE table, DROP table, ...etc
- **Data Manipulation Language** (DML) - SELECT, INSERT, DELETE, UPDATE, ...etc

---
## Relational Model

An efficient model that's used by all major commercial database systems. Can be queried with high-level languages.

- **Database** - set of named *relations* (or *tables*)
- **Relations/tables** - set of named *attributes* (or *columns*)
- **Tuples/rows** - has a *value* for each *attribute*
- **Attributes** - have a type

---
## SQL Constraints

SQL constraints are used to specify rules for the data in a table and to ensure data integrity.

Constraints are used to limit the type of data that can go into a table. This ensures the accuracy and reliability of the data in the table. If there is any violation between the constraint and the data action, the action is aborted.

Constraints can be column level or table level. Column level constraints apply to a column, and table level constraints apply to the whole table.

- _PRIMARY KEY_ - A combination of a NOT NULL and UNIQUE. Uniquely identifies each row in a table
- _FOREIGN KEY_ - Prevents actions that would destroy links between tables
- _NOT NULL_ - Ensures that a column cannot have a NULL value
- _UNIQUE_ - Ensures that all values in a column are different
- _CHECK_ - Ensures that the values in a column satisfies a specific condition
- _DEFAULT_ - Sets a default value for a column if no value is specified
- _CREATE INDEX_ - Used to create and retrieve data from the database very quickly

---
## SQL Joins

SQL joins are used to **combine rows from two or more tables** based on a related column (usually a foreign key).

- **INNER JOIN**
  - Returns only matching rows from both tables.
  - Most common type.
  - “Intersection” / Cartesian product of both tables.
- **OUTER JOIN** (or FULL OUTER JOIN)
  - Returns all rows from both tables.
  - If no match, missing side is `NULL`.
  - *LEFT OUTER JOIN* / *RIGHT OUTER JOIN* variants exist.
- **NATURAL JOIN**
  - Automatically joins tables **on all columns with the same name**.
  - No need to specify `ON` clause.
  - Behaves like an `INNER JOIN` with implicit join conditions.

---
## Aggregation

**Aggregation functions** are used in the `SELECT` statement to perform calculations on sets of values across multiple rows in a table.

- **MIN()** — Returns the smallest value.
- **MAX()** — Returns the largest value.
- **SUM()** — Calculates the total sum.
- **AVG()** — Computes the average value.
- **COUNT()** — Counts the number of rows or non-null values.
- **Additional Clauses for Aggregation**
	- **GROUP BY**  
	  Divides rows into groups based on one or more columns, so aggregation functions are applied within each group.
	- **HAVING**  
	  Filters groups based on conditions applied to aggregated values (like a WHERE clause but for groups).

---
## SQL Data Types

- **Numeric Types**  
  - `INT`, `INTEGER` — Whole numbers  
  - `SMALLINT`, `BIGINT` — Smaller or larger integer ranges  
  - `DECIMAL(p, s)`, `NUMERIC` — Fixed precision decimals  
  - `FLOAT`, `REAL`, `DOUBLE` — Approximate floating-point numbers

- **Character/String Types**  
  - `CHAR(n)` — Fixed-length strings  
  - `VARCHAR(n)` — Variable-length strings  
  - `TEXT` — Large text blocks (unbounded or very large)

- **Date & Time Types**  
  - `DATE` — Calendar date (year, month, day)  
  - `TIME` — Time of day (hours, minutes, seconds)  
  - `TIMESTAMP` — Date and time combined  
  - `INTERVAL` — Time intervals/durations (in some DBs)

- **Boolean Type**  
  - `BOOLEAN` — True/False values

- **Binary Types** (for raw bytes)  
  - `BLOB`, `BYTEA`, `BINARY` — Store binary data like images, files

---