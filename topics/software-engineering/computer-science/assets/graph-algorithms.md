# Graph Algorithms

#deep-dive

Graph algorithms operate on graph data structures to solve problems involving relationships and connectivity. They enable efficient traversal, pathfinding, and cycle detection across nodes and edges, forming the core of network analysis, routing, and optimization tasks.

---
### Breadth-First Search

Breadth-first search for a graph is a way to traverse the graph. 

It starts at the root node and explores all of the neighbor nodes at the present depth prior to moving on to the nodes at the next depth level.

```JAVA
 public static List<String> bfs(Map<String, List<String>> graph, String start) {
        Set<String> visited = new HashSet<>();
        Queue<String> queue = new LinkedList<>();
        List<String> result = new ArrayList<>();

        visited.add(start);
        queue.offer(start);

        while (!queue.isEmpty()) {
            String node = queue.poll();
            result.add(node);

            for (String neighbor : graph.getOrDefault(node, Collections.emptyList())) {
                if (!visited.contains(neighbor)) {
                    visited.add(neighbor);
                    queue.offer(neighbor);
                }
            }
        }

        return result;
    }
```

---
### Depth-First Search

Depth first search is a graph traversal algorithm that starts at a root node and explores as far as possible along each branch before backtracking.

```JAVA
 public static List<String> dfs(Map<String, List<String>> graph, String start) {
        Set<String> visited = new HashSet<>();
        List<String> result = new ArrayList<>();
        dfsHelper(graph, start, visited, result);
        return result;
    }

    private static void dfsHelper(Map<String, List<String>> graph, String node, Set<String> visited, List<String> result) {
        if (visited.contains(node)) return;

        visited.add(node);
        result.add(node);

        for (String neighbor : graph.getOrDefault(node, Collections.emptyList())) {
            dfsHelper(graph, neighbor, visited, result);
        }
    }
```

---
### Bellman Ford's Algorithm

Bellman Ford's algorithm is a graph algorithm that finds the shortest path from a source vertex to all other vertices in a graph. 

It is a dynamic programming algorithm that uses a bottom-up approach to find the shortest path. It is similar to Dijkstra's algorithm but it can handle negative weights. It is also similar to Floyd-Warshall's algorithm but it can handle negative weights and it is faster than Floyd-Warshall's algorithm.

[Bellman-Ford - MIT](https://www.youtube.com/watch?v=f9cVS_URPc0&ab_channel=MITOpenCourseWare)

---
### Dijkstra's Algorithm

Dijkstra's algorithm is a graph traversal algorithm that finds the shortest path between two nodes in a graph. It is a weighted graph algorithm, meaning that each edge in the graph has a weight associated with it. 

The algorithm works by finding the shortest path from the starting node to all other nodes in the graph. It does this by keeping track of the distance from the starting node to each node, and then choosing the node with the shortest distance from the starting node to visit next. It then updates the distance of each node from the starting node, and repeats the process until all nodes have been visited.

[Dijkstras Algorithm - MIT](https://www.youtube.com/watch?v=NSHizBK9JD8&t=1731s&ab_channel=MITOpenCourseWare)

---
### A* Algorithm

A* is a graph traversal algorithm that is used to find the shortest path between two nodes in a graph. It is a modified version of Dijkstra's algorithm that uses heuristics to find the shortest path. It is used in pathfinding and graph traversal.

