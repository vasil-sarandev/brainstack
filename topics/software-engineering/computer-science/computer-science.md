# Computer Science

#topic 

---
## Resources

- **Deep Dives**
	- [Trees](assets/trees.md)
	- [Tree Algorithms](assets/tree-algorithms.md)
	- [Graphs](assets/graphs.md)
	- [Graph Algorithms](assets/graph-algorithms.md)
	- [Sorting Algorithms](assets/sorting-algorithms.md)

- **Other Resources** 
	- [Computer Science Roadmap - roadmap.sh](https://roadmap.sh/computer-science) 
	- https://bradfieldcs.com/algos/ (DS and Algorithms Reference)

---
## Data Structures 

### Arrays

Store elements in contiguous memory locations.

- Fast random access: **O(1)**. 
- Slow insertions/deletions (except at end): **O(n)**.
- Use when frequent indexing is required.

### LinkedLists

Stores its items in "containers". The list has a link to the first container and each container has a link to the next container in the list.

- **Types**:
	- **Singly**: one pointer to next.
	- **Doubly**: pointers to next and previous.
	- **Circular**: last node points to head.
- Fast insert/delete at head/tail: **O(1)** (with pointers).
- Slow search: **O(n)**.
- Use when frequent insertions/deletions are needed.

### Stack

 A linear collection of items where items are inserted and removed in a particular order (LIFO - Last In, First Out).
 
 - Operations:
	- `push()` → add to top
	- `pop()` → remove from top
	- `peek()` → view top
- Time Complexity: **O(1)** for all operations.
- Use for: expression parsing, undo operations, backtracking.

### Queue

A linear collection of items where items are inserted and removed in a particular order (FIFO - First in, First Out).

- Operations:
	- `enqueue()` → add to rear
	- `dequeue()` → remove from front
	- `peek()` → view front
- Time Complexity: **O(1)**.
- Variants: Circular Queue, Dequeue, Priority Queue.
- Use for: task scheduling, BFS traversal.

### Hash Map / Hash Table

`Hash Table`, `Map`, `HashMap`, `Dictionary` or `Associative` are all the names of the same data structure.

- Stores data as **(key, value)** pairs.
- Average case: **O(1)** for insert/search/delete.
- Worst case (with collisions): **O(n)**.
- Use for: fast lookups, caching, counting frequencies.

### Trees

A hierarchical data structure consisting of a collection of nodes such that each node of the tree stores a value and a list of references to other nodes (the “children”).

#### Binary Search Tree (BST)

Every node is bigger than its left subtree and smaller than its right subtree. A BST can be generated from any collection of numbers and its layout depends on the order of insertion.

- Left < Node < Right  
- Avg search/insert/delete: **O(log n)**

#### Special Tree Types
| Type                 | Notes                                                  |
| -------------------- | ------------------------------------------------------ |
| Full Binary Tree     | Each node has 0 or 2 children                          |
| Complete Binary Tree | All levels filled except possibly last (left to right) |
| Balanced Tree        | Height balanced (e.g. AVL, Red-Black Trees)            |
#### Heap

- Complete binary tree with heap property.
	- **Min Heap**: every node is smaller than its children with root being the smallest.
	- **Max Heap**: every node is bigger than its children with root being the biggest.

#### Balanced Binary Search Trees (Self-Balancing)

These are some of the types of trees that exist to keep a BST balanced which helps with keeping the time complexity of operations closer to logarithmic `O(log n)` rather than linear `O(n)`.

| Type      | Characteristics                                                 |
| --------- | --------------------------------------------------------------- |
| AVL       | Strict balancing → faster lookups, slower inserts/deletes       |
| Red-Black | Looser balancing → better for inserts/deletes, used in Java/Map |
| 2-3 Tree  | Variable node size → faster inserts, rare in practice           |
| B-Tree    | Used in databases and filesystems for disk-optimized access     |
| K-D Tree  | Fast nearest neighbor in high-dimensional space (used in ML/CV) |
#### Tries

A tree-like data structure that can be used to store strings. The idea is to store the characters of the string in such a way that each node of the tree represents a single character and each vertex represent a single word or a prefix.

### Graphs

Non-linear data structures made up of a finite number of nodes or vertices and the edges that connect them. 

- **Directed** graphs - edges have direction
- **Undirected** graphs - edges are bidirectional
- **Weighted** graphs - edges have weights - (costs, distance, ...)
- **Representations**
	- **Adjacency Matrix** - 2D array `V x V`
	- **Adjacency List** - array of lists (space-efficient)
- Use for: pathfinding (Dijkstra, A*), network modeling, dependency resolution

### Time Complexity Summary

| Data Structure | Access   | Search   | Insertion | Deletion |
| -------------- | -------- | -------- | --------- | -------- |
| Array          | O(1)     | O(n)     | O(n)      | O(n)     |
| Linked List    | O(n)     | O(n)     | O(1)*     | O(1)*    |
| Stack          | O(n)     | O(n)     | O(1)      | O(1)     |
| Queue          | O(n)     | O(n)     | O(1)      | O(1)     |
| Hash Table     | N/A      | O(1)     | O(1)      | O(1)     |
| BST (avg)      | O(log n) | O(log n) | O(log n)  | O(log n) |
| BST (worst)    | O(n)     | O(n)     | O(n)      | O(n)     |

--- 
## Asymptotic Notation

Asymptotic notation is used to describe the efficiency of an algorithm as the input size grows. It helps compare algorithms based on their **growth rate**, abstracting away constants and lower-order terms.
### **Notations** 

- **Big O Notation (`O`)**  - describes the **upper bound** of an algorithm's running time. It tells us the **worst-case** scenario as input size `n` grows.
- **Big Omega Notation (`Ω`)** - describes the **lower bound**, i.e., the **best-case** performance of an algorithm.
- **Big Theta Notation (`Θ`)**  - describes the **tight bound**. It means the algorithm’s runtime is both **upper and lower bounded** by the same function — i.e., it always runs in Θ(f(n)) time.

### Common Time Complexities

| Name             | Complexity | Description                                                                                                                                               |
| ---------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Constant Time    | `O(1)`     | Always takes the same amount of time regardless of input size. Example: array access.                                                                     |
| Logarithmic Time | `O(log n)` | Cuts the problem size by a factor each step. Example: binary search. <br>Base is usually 2, meaning ~10 steps for `n = 1024`, ~5 steps for `n = 32`, etc. |
| Linear Time      | `O(n)`     | Runtime grows proportionally with input size. Example: iterating over an array.                                                                           |
| Polynomial Time  | `O(n^k)`   | General polynomial growth. Example: bubble sort is `O(n^2)`, matrix multiplication is `O(n^3)`.                                                           |
| Exponential Time | `O(2^n)`   | Runtime doubles with each additional input. Example: solving subsets recursively.                                                                         |
| Factorial Time   | `O(n!)`    | Extremely inefficient and should be avoided. Example: brute-force traveling salesman problem.                                                             |

---
## Common Algorithms

### Sorting Algorithms

| Algorithm      | Time Complexity | Description                                           |
| -------------- | --------------- | ----------------------------------------------------- |
| Bubble Sort    | O(n²)           | Repeatedly swaps adjacent elements if out of order    |
| Selection Sort | O(n²)           | Selects the minimum and moves it to the front         |
| Insertion Sort | O(n²)           | Inserts each element into its correct position        |
| Heap Sort      | O(n log n)      | Uses a heap to repeatedly extract the root            |
| Quick Sort     | O(n log n) avg  | Partitions around pivot; sorts partitions recursively |
| Merge Sort     | O(n log n)      | Divides array, sorts halves, and merges them          |

### Recursion

- **Tail Recursion**: Recursive call is the final step (optimizable in some languages).
- **Non-Tail Recursion**: Work is done after the recursive call.

### Tree Traversal Algorithms

- **In-Order**: Left → Root → Right (gives us the sorted array when traversing a BST)  
- **Pre-Order**: Root → Left → Right  
- **Post-Order**: Left → Right → Root  
- **BFS (Level Order)**: Uses a queue to explore level-by-level  
- **DFS**: Uses a stack or recursion to explore deeply. Can be implemented with In-Order, Post-Order, or Pre-Order traversal.

```JAVASCRIPT

const inOrder = (node) => {
	if (node === null) return;
    inOrder(node.left)     ← go left
    visit(node)            ← ADD to array (when we are done with left subtree)
    inOrder(node.right)    ← go right
}

const preOrder = (node) => {
	if (node === null) return;
    visit(node)             ← ADD to array here
    preOrder(node.left)     ← then go left
    preOrder(node.right)    ← then go right
}

const postOrder = (node) => {
	if (node === null) return;
    postOrder(node.left)     ← go left
    postOrder(node.right)    ← go right
    visit(node)             ← ADD to array here
}
```

### Graph Algorithms

- **BFS** – Explores neighbors level-by-level  
- **DFS** – Explores depth before backtracking  
- **Dijkstra’s Algorithm** – Shortest path with priority queue (no negative weights)  
- **A\*** – Like Dijkstra, but uses a heuristic to guide the search  
- **Bellman-Ford** – Handles negative weights; slower  
- **Kruskal’s Algorithm** – Greedy MST using Union-Find  
- **Prim’s Algorithm** – Greedy MST starting from a node  
- **Ford-Fulkerson** – Max flow using augmenting paths (BFS/DFS)

### Search Algorithms

| Algorithm     | Time Complexity | Description                                |
| ------------- | --------------- | ------------------------------------------ |
| Binary Search | O(log n)        | Halves the search space (on sorted arrays) |

### Cache Eviction Policies

| Policy                          | Description                             |
| ------------------------------- | --------------------------------------- |
| **LRU** (Least Recently Used)   | Evicts the least recently accessed item |
| **LFU** (Least Frequently Used) | Evicts the item used least often        |
| **MRU** (Most Recently Used)    | Evicts the most recently accessed item  |

### Backtracking Problems

- **N-Queens** – Place queens so none attack each other  
- **Maze Solver** – Explore paths and backtrack at dead ends  
- **Knight’s Tour** – Visit all chessboard squares with knight moves  
- **Hamiltonian Path** – Visit every vertex exactly once

### Greedy Algorithms

- **Huffman Coding** – Builds optimal prefix codes for compression  
- **Dijkstra’s Algorithm** – Greedy choice of nearest node  
- **Kruskal’s & Prim’s MST** – Greedy edge selection  
- **Ford-Fulkerson** – Greedy selection of augmenting paths

---
## Floating-point numbers

Floating-point numbers are numbers that have a decimal point in them. They are used to represent real numbers. For example, *3.14* is a floating point number. They follow the **IEEE 754 standard**, which defines how numbers like `0.1`, `2.5`, or `3.14159` are stored in memory.

```JAVASCRIPT
// how the computer stores the floating number in memory
0.1 ≈ 0.0001100110011... (repeating)
0.2 ≈ 0.001100110011... (repeating)

// so when added the result is a tiny bit off
0.1 + 0.2 === 0.30000000000000004
```

---
## Endianness

**Endian** and **endianness** (or "byte-order") describe how computers organize the bytes that make up numbers.

Let's use the number `0x12345678` (i.e., 305 419 896 in decimal) for an example:

- **Little-Endian**: the least significant byte is stored first. ex: `0x78 0x56 0x34 0x12`.  It is the most common ordering of multiple bytes.
- **Big-Endian**: the most significant byte is stored first. ex: `0x12 0x34 0x56 0x78`. It is the opposite of Little-Endian

---
## Character Encodings

Character encodings are a way of representing characters as numbers. They are used to store and transmit text. 

- **ASCII** - The most common character encoding is ASCII, which is a 7-bit encoding. This means that each character is represented by a number between 0 and 127.
- **Unicode** - The ASCII character set is a subset of the Unicode character set, which is a 16-bit encoding. It is the most common character encoding used on the web and contains 2^16 characters.

---
## Bitwise Operators

Bitwise operators are used to perform operations on individual bits of a number. They are used in cryptography, image processing, and other applications.

[Algorithms: Bit Manipulation](https://www.youtube.com/watch?v=NLKQEOgBAnw)

---
## Complexity Classes

In complexity theory, a Complexity Class is a set of problems with related complexity. These classes help scientists to groups problems based on how much time and space they require to solve problems and verify the solutions.

### **Deterministic vs Non-Deterministic** 

- **Deterministic** - at each step there's only one choice of what to do next. This is how real-world computers work.
- **Non-deterministic** - at some steps when there's multiple options the machine magically picks the *right path* - the one that leads to the "yes" answer - if such a path exists.

### **Complexity Classes** 

- **P** - The P in the *P* class stands for Polynomial Time. It is the collection of decision problems that can be solved by a deterministic machine in polynomial time - `O(n^k)`
- **NP** - The NP in NP class stands for Non-deterministic Polynomial Time. It is the collection of decision problems that can be solved by a non-deterministic machine in polynomial time. Technically covers problems that can easily be verified in polynomial time but finding the solution for them might take a lot longer.
- **Co-NP** - NP is about problems where we can verify a "Yes" answer quickly, Co-NP is the opposite - we can verify a "No" answer quickly (in polynomial time).
- **NP-complete** - NP-Complete is a subset of *NP* that represents the hardest problems in the complexity class.
- **NP-hard** - At least as hard as the hardest problems in NP (could not be in NP itself i.e not verifiable in polynomial time).

### **P = NP ?** 

The P = NP is a very famous problem in CS that asks whether a problem that can be solved in polynomial time on a non-deterministic machine(i.e the problem is in *NP*) can also be solved in polynomial time on a deterministic machine (i.e the problem is in *P*). 

If it was proven that P=NP, it would revolutionize computing because cryptography and hashing would be easily crackable

---
## Processes and Threads

**Process** is an instance of a program that is being executed.

**Thread** is the smallest unit of execution within a process. Threads within the same process share memory and resources but can execute independently.

### Common Concepts

- **Process Forking** – A way to create new processes from an existing process.
- **Memory Management** – The process of allocating and deallocating memory.
- **Garbage Collection** – Automatically detects and frees memory allocated to objects no longer in use.
- **Reference Counting** – Tracks how many references point to a memory location to determine when it can be safely deallocated.
- **Memory Pools** – Technique to automatically manage memory allocation based on application lifecycle (e.g., request or transaction).

### Lock, Mutex, Semaphore

- **Lock** – Restricts access to a critical section to one thread at a time within the same process.
- **Mutex** – A system-wide lock that can synchronize access across threads and processes.
- **Semaphore** – Allows a limited number (`n`) of threads/processes to access a resource concurrently. Useful for managing resource pools.

### Concurrency and Parallelism

- **Concurrency** – Managing multiple tasks at the same time (interleaving), not necessarily simultaneously. Ideal for I/O-bound operations.
- **Parallelism** – Executing multiple tasks at the exact same time. Beneficial for CPU-bound computations.

> A **concurrent program** can run on a single core using context switching, while a **parallel program** requires multiple cores or processors.

### CPU Scheduling

CPU Scheduling is the process of selecting a process from the ready queue and allocating the CPU to it based on a scheduling algorithm.

**Common Scheduling Algorithms:**

- **Round Robin (RR):** Each process gets a fixed time slice (e.g., 10 ms). Preemptive.
- **First Come First Serve (FCFS):** The earliest arriving process runs first. Non-preemptive.
- **Shortest Job First (SJF):** The process with the shortest execution time is selected. Non-preemptive.
- **Shortest Remaining Time First (SRTF):** Like SJF but preemptive, considering remaining time.
- **Priority Scheduling:** The process with the highest priority runs first. Can be preemptive or non-preemptive.
- **Lottery Scheduling:** Processes receive a chance to run based on randomly drawn tickets. Preemptive.

### Multithreading vs Multiprocessing

- **Multithreading**: Multiple threads within the same process. Shared memory space. Lightweight. Best for **I/O-bound** tasks.
- **Multiprocessing**: Multiple independent processes. Separate memory. More overhead. Best for **CPU-bound** tasks.

> Threads are faster to create and communicate but risk shared state issues. Processes are isolated and safer but more resource-intensive.

---
## How Computers Work

### CPU Execution Cycle

The CPU runs programs by repeatedly performing the **fetch-decode-execute** cycle:

  1. **Fetch**: Retrieve the next instruction from memory.
  2. **Decode**: Interpret the instruction.
  3. **Execute**: Perform the required operation. 
  
  Modern CPUs use **pipelining** and **caches** to speed up execution.

### Registers and Memory

- **Registers**: Small, fast storage inside the CPU holding instructions or data currently in use.
- **RAM (Random Access Memory)**: Primary memory that stores programs and data for quick access by the CPU.
- **Secondary Storage**: Persistent storage like hard drives or SSDs, slower but retains data when powered off.
- **Memory hierarchy**: (Registers > Cache > RAM > Secondary Storage) balances speed and capacity for efficient data access.

### CPU Cache

- Small, very fast memory located close to the CPU cores.
- Stores frequently accessed data to reduce time spent fetching from RAM.

### Binary and Computation

- Computers represent all data in **binary** (0s and 1s), reflecting transistor states (ON/OFF).
- Logic gates (AND, OR, NOT) manipulate binary data to perform calculations.
- Complex numbers and decimals use **floating-point representation**.

---
