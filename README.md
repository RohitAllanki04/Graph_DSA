# 📊 Graph — Concepts & Topological Sort

---

## 🔷 What is a Graph?

A **Graph** is a non-linear data structure consisting of:

- **Vertices (Nodes)** — the entities
- **Edges** — the connections between vertices

```
    A ——— B
    |     |
    C ——— D
```

### Types of Graphs

| Type | Description |
|------|-------------|
| **Undirected** | Edges have no direction (A — B means A↔B) |
| **Directed** | Edges have direction (A → B, not B → A) |
| **Weighted** | Edges carry a cost/weight |
| **Unweighted** | All edges are equal |
| **Cyclic** | Contains at least one cycle |
| **Acyclic** | No cycles exist |

---

## 🔷 Key Graph Concepts

### 📌 In-Degree & Out-Degree
- **In-degree** → Number of **incoming** edges to a vertex
- **Out-degree** → Number of **outgoing** edges from a vertex

```
A → B → C
        ↑
        D

In-degree of C = 2 (from B and D)
```

### 📌 Adjacency List
Representation of a graph using a list of neighbors for each vertex.

```java
// Example
0 → [1, 2]
1 → [3]
2 → [3]
3 → []
```

### 📌 BFS (Breadth First Search)
- Explores neighbors level by level
- Uses a **Queue**

### 📌 DFS (Depth First Search)
- Explores as deep as possible before backtracking
- Uses a **Stack** (or recursion)

---

## 🔷 DAG — Directed Acyclic Graph

A **DAG** is a **Directed** graph with **No Cycles**.

```
A → B → D
↓       ↑
C ———————
```

### ✅ Why DAG?

> **Because cycle dependency causes infinite loops!**

In real-world problems like:
- **Build systems** (compile A before B)
- **Task scheduling** (do task A before task B)
- **Course prerequisites**

If there's a **cycle** (A depends on B, B depends on A), the system breaks. DAG **prevents** this.

---

## 🔷 Topological Sort(https://www.geeksforgeeks.org/problems/topological-sort/1)

**Topological Sort** gives a **linear ordering** of vertices such that for every directed edge `u → v`, vertex `u` comes **before** `v`.

> ⚠️ Only possible on a **DAG** (no cycles)

---

## 🟦 Approach 1: BFS — Kahn's Algorithm

### 💡 Core Idea
Use **in-degree** of each vertex. Process nodes with `in-degree = 0` first (no dependencies).

### 📋 Steps

```
1. Find in-degree (number of incoming edges) of each vertex
2. Add all vertices with in-degree = 0 into a Queue
3. While Queue is not empty:
      a. Dequeue a vertex → add it to result list
      b. For each adjacent (neighbor) vertex:
            - Decrease its in-degree by 1
            - If in-degree becomes 0 → Enqueue it
4. Return the result list
```

### 🖼️ Example

```
Graph:
5 → 0 ← 4
↓         ↓
2 → 3 → 1

In-degrees: { 0:2, 1:1, 2:1, 3:1, 4:0, 5:0 }

Queue starts with: [4, 5]   ← in-degree = 0

Step 1: Process 4 → result=[4], reduce in-degree of 0,1
Step 2: Process 5 → result=[4,5], reduce in-degree of 0,2
Step 3: Process 0 → result=[4,5,0]  (in-degree of 0 became 0)
...and so on
```

### 💻 Java Code

```java
public List<Integer> topoSortBFS(int V, List<List<Integer>> adj) {
    int[] inDegree = new int[V];

    // Step 1: Calculate in-degree of each vertex
    for (int i = 0; i < V; i++)
        for (int neighbor : adj.get(i))
            inDegree[neighbor]++;

    // Step 2: Add all 0 in-degree vertices to queue
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < V; i++)
        if (inDegree[i] == 0)
            queue.offer(i);

    List<Integer> result = new ArrayList<>();

    // Step 3: Process queue
    while (!queue.isEmpty()) {
        int node = queue.poll();
        result.add(node);                        // add to result

        for (int neighbor : adj.get(node)) {
            inDegree[neighbor]--;                // decrease in-degree
            if (inDegree[neighbor] == 0)
                queue.offer(neighbor);           // if 0, enqueue
        }
    }

    return result;
}
```

### ⏱️ Complexity

| | BFS Topological Sort |
|---|---|
| **Time** | O(V + E) |
| **Space** | O(V) |

---

## 🟩 Approach 2: DFS — Stack-Based

### 💡 Core Idea
Run DFS and **push a vertex onto the stack only after all its neighbors are visited** (post-order). Then reverse the stack to get topological order.

### 📋 Steps

```
1. Create a Stack and a visited[] boolean array
2. For each vertex (0 to V-1):
      If NOT visited → call dfs(vertex)
3. Inside dfs(vertex):
      a. Mark vertex as visited
      b. For each unvisited neighbor → recurse dfs(neighbor)
      c. After all neighbors done → push vertex onto Stack
4. Pop all elements from Stack → that's the topological order
```

### 🖼️ Visual Walkthrough

```
Graph: 5→2, 5→0, 4→0, 4→1, 2→3, 3→1

DFS from 5:
  Visit 5 → Visit 2 → Visit 3 → Visit 1
  1 has no unvisited neighbors → push 1
  Back to 3 → push 3
  Back to 2 → push 2
  Back to 5 → push 0 (via 5→0) → push 5

Stack (bottom → top): [1, 3, 2, 0, 5, 4]
Reversed: 4 5 0 2 3 1  ✅
```

### 💻 Java Code

```java
public int[] topoSortDFS(int V, List<List<Integer>> adj) {
    boolean[] visited = new boolean[V];
    Stack<Integer> stack = new Stack<>();

    // Step 1: Call DFS for every unvisited vertex
    for (int i = 0; i < V; i++)
        if (!visited[i])
            dfs(i, visited, stack, adj);

    // Step 2: Pop stack into result array
    int[] result = new int[V];
    int idx = 0;
    while (!stack.isEmpty())
        result[idx++] = stack.pop();

    return result;
}

private void dfs(int node, boolean[] visited,
                 Stack<Integer> stack, List<List<Integer>> adj) {
    visited[node] = true;                        // mark visited

    for (int neighbor : adj.get(node))
        if (!visited[neighbor])
            dfs(neighbor, visited, stack, adj);  // recurse deeper

    stack.push(node);                            // push AFTER all neighbors
}
```

### ⏱️ Complexity

| | DFS Topological Sort |
|---|---|
| **Time** | O(V + E) |
| **Space** | O(V) — stack + visited array |

---

## 🔷 BFS vs DFS Topological Sort

| Feature | BFS (Kahn's) | DFS (Stack) |
|---------|-------------|-------------|
| **Approach** | In-degree based | Post-order recursion |
| **Data Structure** | Queue | Stack + Recursion |
| **Cycle Detection** | ✅ Easy (result size < V) | ❌ Needs extra logic |
| **Intuition** | Process no-dependency nodes first | Go deep, then record |
| **Time Complexity** | O(V + E) | O(V + E) |

---
```
// edges = [[2,1,1],[2,3,1],[3,4,1]]
// n = 4

List<List<int[]>> adj = new ArrayList<>();
for(int i = 0; i <= n; i++)
    adj.add(new ArrayList<>());

for(int[] edge : edges) {
    adj.get(edge[0]).add(new int[]{edge[1], edge[2]});//single - directed graph
    adj.get(edge[1]).add(new int[]{edge[0], edge[2]});//add if bi-directed graph
}

📊 What gets stored
Input: [[2,1,1],[2,3,1],[3,4,1]]

adj[0] = []
adj[1] = [[2,1]]
adj[2] = [[1,1],[3,1]]
adj[3] = [[2,1],[4,1]]
adj[4] = [[3,1]]

```

