# NodeJS Cheatsheet

#language 

Node.js is a runtime environment that allows you to run JavaScript on the server side. Built on Chrome’s V8 engine, it enables fast, event-driven, non-blocking I/O operations, making it ideal for scalable network applications

Core features:

- **Event-driven, non-blocking I/O** for high concurrency without threading overhead.
- **Single-threaded** but can handle many connections simultaneously via the event loop.
- **Built-in modules** for networking, file system, streams, and more.

---
## Resources

- **Deep Dives**
	- [Event Loop - NodeJS Reference](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)
	- [Don't Block the Event Loop](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)
	- [NodeJS Project Architecture Best Practices](https://github.com/goldbergyoni/nodebestpractices#1-project-architecture-practices)

- **Hands-on**
	- [Reading Files with NodeJS](https://nodejs.org/en/learn/manipulating-files/reading-files-with-nodejs)
	- [How to use Streams](https://nodejs.org/en/learn/modules/how-to-use-streams) 
	- [Writing Files with NodeJS](https://nodejs.org/en/learn/manipulating-files/writing-files-with-nodejs)
	- [Read Input from Command Line in NodeJS](https://nodejs.org/en/learn/command-line/accept-input-from-the-command-line-in-nodejs)
	- [Setting up a NodeJS project with TypeScript - DigitalOcean](https://www.digitalocean.com/community/tutorials/setting-up-a-node-project-with-typescript)

- **Docs & References**
	- [Learn NodeJS - Official Docs](https://nodejs.org/en/learn/)
	- [Github - Node Best Practices](https://github.com/goldbergyoni/nodebestpractices) - This one is awesome and has a lot of info about architecture, code style, production practices, and etc.
	- [Github - Vasil Sarandev - Node Starter Kit](https://github.com/vasil-sarandev/node-starter)

---
## Blocking vs Non-Blocking

> "I/O" refers primarily to interaction with the system's disk and network supported by [libuv](https://libuv.org/).

**Blocking** is when the execution of additional JavaScript in the Node.js process must wait until a non-JavaScript operation completes. This happens because the event loop is unable to continue running JavaScript while a **blocking** operation is occurring.

All of the I/O methods in the Node.js standard library provide asynchronous versions, which are **non-blocking**, and accept callback functions. Some methods also have **blocking** counterparts, which have names that end with `sync`.

---
## Event Loop

The Event Loop is what enables Node.js to perform non-blocking I/O operations (such as database reads, file system access, and HTTP requests) — even though JavaScript itself runs on a single thread. It does this by offloading I/O operations to the system kernel or a worker thread pool whenever possible.

**How the Event Loop works (simplified)**:

1. A request or event is received (e.g., an incoming HTTP request).
2. If the operation is asynchronous, it’s registered and dispatched to `libuv`.
3. For eligible operations, `libuv` delegates them either to the system kernel (if they are natively non-blocking) or to its internal thread pool.
4. Once the operation completes, a callback is pushed to the Event Queue (also called the Callback Queue).
5. The Event Loop continuously checks the queue and, when the call stack is clear, pushes callbacks onto it for execution.

---
## Event Emitter

Much of what the JavaScript code in browser reacts to is user events.

On the backend side, Node.js offers us the option to build a similar system using the *events module* (node:events). 

It's typically used in low-level modules and NodeJS Core APIs - `http.Server`, `fs.ReadStream`, `net.Socket` all extend the `EventEmitter` class.

```javascript
eventEmitter.on('start', () => {...});
eventEmitter.emit('start');
```

---
## Cluster and Worker Threads

Node.js runs JavaScript on a single thread by default, but to leverage multiple CPU cores and perform parallel processing, Node.js provides:

- **Cluster module** — for scaling across multiple processes.
- **Worker Threads** — for parallelism within the same process.

### When to Use What?

The **Cluster** module allows you to create multiple Node.js processes (workers) that share the same server port. This is useful to take advantage of multi-core systems and improve throughput.

**Worker Threads** enable running JavaScript code in parallel threads inside the same Node.js process. They share memory via `SharedArrayBuffer` but have isolated event loops. Use worker threads for CPU-intensive tasks that would block the main event loop.

---
## Layered Architecture in NodeJS

Keeps concerns separated, improves testability and maintainability.

- **Controller**: Handles HTTP requests, calls services, sends responses.
- **Service**: Contains business logic, coordinates data and operations.
- **Repository**: Handles database access and raw queries.

---




