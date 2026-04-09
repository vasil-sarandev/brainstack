# TypeORM

#technology

TypeORM is an Object-Relational Mapper (ORM) for Node.js written in TypeScript and JavaScript. It supports various databases (PostgreSQL, MySQL, MariaDB, SQLite, SQL Server, Oracle, MongoDB, etc.) and provides a clean, type-safe way to interact with relational data using entities, repositories, and data mappers.

---
## Resources

- **Deep Dives**
	- [Migrations - TypeORM][https://typeorm.io/docs/advanced-topics/migrations]
	- [Transactions - TypeORM](https://typeorm.io/docs/advanced-topics/transactions)
	- [Indices - TypeORM](https://typeorm.io/docs/advanced-topics/indices)
	- [Performance Optimizations - TypeORM](https://typeorm.io/docs/advanced-topics/performance-optimizing)

- **Docs and References**
	- [TypeORM Official Documentation](https://typeorm.io/#/)
	- [Awesome TypeORM (Community Curated)](https://github.com/typeorm/typeorm)

---
## Core Concepts

- **Entity** - A class mapped to a database table. Decorated with `@Entity()`, with fields as columns.
- **Column** - A property in an entity mapped to a table column. Decorated with `@Column()` and supports options like `type`, `length`, `nullable`, etc.
- **Primary Column** - A unique identifier for each row. Declared using `@PrimaryColumn()` or `@PrimaryGeneratedColumn()`.
- **Repository** - Handles database operations for an entity.
- **DataSource** - Manages connection to the database and provides access to repositories.
- **Relations** - Define connections between entities. Relations must be defined on **both** the owning and inverse sides to enable bidirectional navigation, but **only one side** owns the relationship and manages the foreign key or join table.
	- `@OneToOne` – The owner side **must** be defined with the `@JoinColumn` decorator. The owner entity contains the foreign key column referencing the related entity.
	- `@OneToMany` – This is the **inverse side** of a one-to-many relation; it does **not** own the relation and does **not** have a foreign key column. The owner side is the corresponding **`@ManyToOne`** side.
	- `@ManyToOne` – The owner side is **always the many side**; it contains the foreign key column.
	- `@ManyToMany` – The owning side is defined with the `@JoinTable` decorator, which creates a junction table. Only one side can be the owner.
- **Migrations** - Version-controlled scripts for schema changes. Run with TypeORM CLI or programmatically.
- **QueryBuilder** - Flexible API for building SQL queries.
```ts
	  const users = await userRepo.createQueryBuilder("user")
		.where("user.isActive = :isActive", { isActive: true })
		.getMany();
```
- **SQL** - TypeORM provides a way to write SQL queries using template literals with automatic parameter handling based on your database type.
```typescript
const users = await dataSource.sql`SELECT * FROM users WHERE name = ${"John"}`
```

---
## Active Record vs Data Mapper

TypeORM supports two architectural patterns for working with entities.

 - **Active Record Pattern** 
	**Entities contain both data and methods** for database operations: 
	- Calls like `save()`, `remove()`, etc. are made directly on entity instances.
	- Simple and straightforward for small apps but harder to test in large codebases.
-  **Data Mapper** 
	**Entities are plain objects** with no persistence logic. **Repositories handle all DB operations*.
	- Clear separation of concerns, better for complex apps.
	- Slightly more boilerplate.

---

## Find Options

- **Strings**
	- Contains: `ILike("%word%")` / SQL: `WHERE col ILIKE '%word%'`
	- Starts with: `ILike("word%")` / SQL: `WHERE col ILIKE 'word%'`
	- Ends with: `ILike("%word")` / SQL: `WHERE col ILIKE '%word'`

```TypeScript
import { ILike } from "typeorm";

const users = await userRepository.find({ where: { firstName: ILike("%alex%") } });
```

- **Numbers**
	- **Greater Than:** `MoreThan(10)`
	- **Less Than:** `LessThan(10)`
	- **Inclusive:** `MoreThanOrEqual(10)` or `LessThanOrEqual(10)`
	- **Between:** `Between(1, 10)`

```Typescript
import { MoreThan, Between } from "typeorm";

const products = await productRepository.find({ where: { price: MoreThan(50), stock: Between(10, 100) } });
```

- **Arrays**

```Typescript
import { In, ArrayContains } from "typeorm"; 
// Find users with specific IDs 
const users = await userRepository.find({ where: { id: In([1, 2, 5]) } });
```

- **Logical Operators**

```Typescript
// WHERE firstName = 'Jane' AND isAdmin = true 
const users = await userRepository.find({ where: { firstName: "Jane", isAdmin: true } });
// WHERE firstName = 'Jane' or 'Joe' and isAdmin=true
const users = await userRepository.find({ where: [ { firstName: "Jane", isAdmin: true }, { firstName: "Joe", isAdmin: true } ] });
```

