# Java Cheatsheet

#language 

Java is a high-level, class-based, object-oriented programming language designed to have as few implementation dependencies as possible. It operates on the "Write Once, Run Anywhere" (WORA) principle, meaning compiled Java code can run on all platforms that support Java without the need for recompilation.

### Core features:

- **Platform Independence:** Bytecode runs on the **JVM** (Java Virtual Machine), regardless of the underlying hardware.
- **Strongly Typed:** Catch errors at compile-time rather than runtime.
- **Automatic Memory Management:** The **Garbage Collector (GC)** automatically deallocates memory, preventing most memory leaks.
- **Multithreading:** Built-in support for concurrent execution of tasks.

---

### Resources

 - **Deep Dives**
	- [The Java Virtual Machine (JVM) Specification](https://docs.oracle.com/javase/specs/jvms/se21/html/index.html)
	- [Java Memory Management & Garbage Collection](https://www.oracle.com/technical-resources/articles/java/g1gc.html)
	- [Baeldung - Guide to Java Collections](https://www.baeldung.com/java-collections)
- **Hands-on**
	- [Building a REST API with Spring Boot](https://spring.io/guides/gs/rest-service/)
	- [Java Streams API Explained](https://www.oracle.com/technical-resources/articles/java/ma14-java-se-8-streams.html)
	- [Maven vs Gradle: Which one to choose?](https://www.google.com/search?q=https://gradle.org/maven-vs-gradle/)
- **Docs & References**
	- [Java 21/26 Official Documentation](https://docs.oracle.com/en/java/javase/)
	- [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
	- [Baeldung Java Tutorials](https://www.baeldung.com/) — The "gold standard" for Java developers.

---

### The Runtime (JVM - Java Virtual Machine)

- **Compilation:** Java code (`.java`) is compiled into **Bytecode** (`.class`) via `javac`.
- **JIT Compilation:** The **Just-In-Time (JIT)** compiler turns bytecode into native machine code at runtime for high performance.
- **JRE vs JDK:** You need the **JDK** (Java Development Kit) to develop; the **JRE** (Runtime Environment) is just for running apps.

---

### Package Managers (Build Tools)

In Node, you have `npm`. In Java, you use **Maven** or **Gradle**. They handle dependencies, testing, and packaging (`.jar` files).

- **Maven (`pom.xml`):** XML-based, very structured, and the industry standard for enterprise.
- **Gradle (`build.gradle`):** Script-based (Groovy/Kotlin), faster, and more flexible. Preferred for modern and Android projects.

---

### Web Framework: Spring Boot

The equivalent to **NestJS** or **Express**. It is the most popular framework for Java web development.

- **Inversion of Control (IoC):** The framework manages your objects (Beans) and injects them where needed.
- **Dependency Injection (DI):** Similar to NestJS, you define dependencies in constructors.
- **Spring Initializr:** Use [start.spring.io](https://start.spring.io) to generate a new project.

---

### ORMs

1. **JPA:** The specifications. These are the annotations like `@Entity`, `@Column`, and `@Id` that you put on your classes.
2. **Hibernate:** The engine that translates Java objects into SQL. 
3. **Spring Data JPA:** The interface that gives you `save()`, `delete()`, and `findAll()`

---

### Multithreading & Virtual Threads

Java is multi-threaded by nature.

- **Platform Threads:** Heavyweight, mapped directly to OS threads.
- **Virtual Threads (Project Loom):** Introduced in Java 21. These are lightweight threads (similar to Go routines) that allow you to handle millions of concurrent requests without blocking the OS thread.

---

### Layered Architecture in Java (Spring)

Standard enterprise pattern to keep code clean.

- **Controller (`@RestController`):** Handles incoming HTTP requests and maps them to methods.
- **Service (`@Service`):** The "Brain." Contains business logic and calls repositories.
- **Repository (`@Repository`):** The "Data Layer." Uses Spring Data JPA to talk to the database.
- **Entity (`@Entity`):** The data model representing a database table.
- **DTO (Data Transfer Object):** Simple classes (usually **Records**) used to send data back to the client.