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
---
 
## 🔷 Dijkstra's Algorithm — Shortest Path
 
### 💡 When to Use?
> When you need to find the **shortest path** from a **single source** to all other vertices in a **weighted directed graph**.
 
Real world examples:
- **Network delay** — minimum time for signal to reach all nodes
- **GPS navigation** — shortest route from A to B
- **Flight routes** — cheapest path between cities
---
 
### 💡 Intuition
 
```
Always process the NEAREST unvisited vertex first
→ greedy approach using Min Heap (Priority Queue)
→ once a vertex is processed with minimum distance, it's final
→ keep relaxing neighbors if shorter path found
```
 
---
 
### 📋 Steps
 
```
1. Create a Min Heap (Priority Queue) — stores [vertex, distance]
   → always picks vertex with LEAST distance first
 
2. Create dist[] array — fill with MAX_INTEGER (unreachable)
   → dist[source] = 0 (distance to itself is 0)
 
3. Offer source vertex into queue with distance 0
 
4. While queue is not empty:
      a. Poll [vertex, distance] from queue (minimum distance first)
      b. If current distance > dist[vertex] → skip (outdated entry)
      c. For each neighbor of vertex:
            - newDist = dist[vertex] + edge weight
            - If newDist < dist[neighbor]:
                  → update dist[neighbor] = newDist
                  → offer [neighbor, newDist] into queue
 
5. Finally:
      → dist[] holds shortest distance from source to every vertex
      → if any dist[i] == MAX_INTEGER → vertex unreachable → return -1
      → max of dist[] = minimum time for ALL nodes to receive signal
```
 
---
 
### 🖼️ Visual Walkthrough
 
```
Graph: k=1 (source)
1 →(2)→ 2
1 →(4)→ 3
2 →(1)→ 3
 
dist = [MAX, 0, MAX, MAX]   ← dist[1]=0 (source)
pq   = [{1,0}]
 
Step 1: poll {1,0}
  neighbor 2: dist[1]+2=2 < MAX → dist[2]=2, offer {2,2}
  neighbor 3: dist[1]+4=4 < MAX → dist[3]=4, offer {3,4}
  pq = [{2,2},{3,4}]
 
Step 2: poll {2,2}  ← min distance first!
  neighbor 3: dist[2]+1=3 < dist[3]=4 → dist[3]=3, offer {3,3}
  pq = [{3,3},{3,4}]
 
Step 3: poll {3,3}
  no neighbors
  pq = [{3,4}]
 
Step 4: poll {3,4}
  4 > dist[3]=3 → SKIP (outdated)
 
dist = [MAX, 0, 2, 3]
max  = 3 ✅
```
 
---
 
### 💻 Java Code
 
```java
public int networkDelayTime(int[][] times, int n, int k) {
    // Step 1: Build adjacency list [neighbor, weight]
    List<List<int[]>> adj = new ArrayList<>();
    for(int i = 0; i <= n; i++)
        adj.add(new ArrayList<>());
 
    for(int[] edge : times)
        adj.get(edge[0]).add(new int[]{edge[1], edge[2]});
 
    // Step 2: dist array — fill MAX, source = 0
    int[] dist = new int[n+1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;
 
    // Step 3: Min heap [vertex, distance]
    PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> a[1]-b[1]);
    pq.offer(new int[]{k, 0});
 
    // Step 4: Process queue
    while(!pq.isEmpty()) {
        int[] cur  = pq.poll();
        int node   = cur[0];
        int d      = cur[1];
 
        if(d > dist[node]) continue;       // skip outdated entry
 
        for(int[] nei : adj.get(node)) {
            int neighbor = nei[0];
            int weight   = nei[1];
 
            if(dist[node] + weight < dist[neighbor]) {
                dist[neighbor] = dist[node] + weight;          // update
                pq.offer(new int[]{neighbor, dist[neighbor]}); // offer
            }
        }
    }
 
    // Step 5: Find max of dist[]
    int maxDist = 0;
    for(int i = 1; i <= n; i++) {
        if(dist[i] == Integer.MAX_VALUE) return -1;  // unreachable
        maxDist = Math.max(maxDist, dist[i]);
    }
    return maxDist;
}
```
 
---
 
### ⏱️ Complexity
 
| | Dijkstra |
|---|---|
| **Time** | O((V + E) log V) |
| **Space** | O(V + E) |
 
---
 
### 📊 Key Points to Remember
 
| Concept | Explanation |
|---------|-------------|
| Min Heap | always picks shortest distance vertex first |
| `dist[]=MAX` | means unreachable initially |
| `d > dist[node]` | skip outdated entries in queue |
| Update neighbor | only if new path is shorter |
| Final answer | max of all `dist[]` values |
| Unreachable | if any `dist[i]==MAX` → return -1 |
 
---
 
### ❌ Why not DFS/BFS for shortest path?
 
```
DFS → goes deep, not shortest path, can't compare paths ❌
BFS → works only for UNWEIGHTED graphs ❌
Dijkstra → always finds shortest in WEIGHTED graphs ✅
```
 
---
 
## 🟨 Dijkstra — Alternative using Set (Visited)
 
### 💡 Idea
Instead of `if(d > dist[v]) continue` — use a **Set** to track already processed vertices and skip them.
 
---
 
### ⚠️ Why NOT `Set<int[]>`
 
```java
Set<int[]> set = new HashSet<>();
int[] a = {1, 2};
int[] b = {1, 2};
set.add(a);
set.add(b);  // ← ADDED as duplicate! NOT blocked!
// int[] uses reference equality, not content equality
// so two arrays with same values = different objects
```
 
### ✅ Use `Set<Integer>` — store vertex number only
 
```java
Set<Integer> visited = new HashSet<>();  // ✅ correct
```
 
---
 
### 📊 Set Types — Quick Reference
 
| Set | Order | Speed | Use When |
|-----|-------|-------|----------|
| `HashSet` | none | O(1) | fast lookup, no order needed |
| `LinkedHashSet` | insertion order | O(1) | maintain insertion order |
| `TreeSet` | sorted order | O(log n) | need sorted order |
 
---
 
### 💻 Complete Code — Dijkstra with Set
 
```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        // Build adjacency list
        List<List<int[]>> adj = new ArrayList<>();
        for(int i = 0; i <= n; i++)
            adj.add(new ArrayList<>());
 
        for(int[] edge : times)
            adj.get(edge[0]).add(new int[]{edge[1], edge[2]});
 
        // dist array
        int[] dist = new int[n+1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[k] = 0;
 
        // Min heap [vertex, distance]
        PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> a[1]-b[1]);
        pq.offer(new int[]{k, 0});
 
        // ✅ Set to track processed vertices
        Set<Integer> visited = new HashSet<>();
 
        while(!pq.isEmpty()) {
            int[] cur = pq.poll();
            int node  = cur[0];
            int d     = cur[1];
 
            if(visited.contains(node)) continue;  // ✅ skip already processed
            visited.add(node);                    // ✅ mark as processed
 
            for(int[] nei : adj.get(node)) {
                int neighbor = nei[0];
                int weight   = nei[1];
 
                if(!visited.contains(neighbor)) {             // skip processed
                    if(dist[node] + weight < dist[neighbor]) {
                        dist[neighbor] = dist[node] + weight;
                        pq.offer(new int[]{neighbor, dist[neighbor]});
                    }
                }
            }
        }
 
        // Find max distance
        int maxDist = 0;
        for(int i = 1; i <= n; i++) {
            if(dist[i] == Integer.MAX_VALUE) return -1;
            maxDist = Math.max(maxDist, dist[i]);
        }
        return maxDist;
    }
}
```
 
---
 
### 📊 Two Approaches Compared
 
| | `d > dist[v]` check | `Set<Integer>` visited |
|---|---|---|
| **Skip condition** | outdated distance | already processed node |
| **How it works** | compare weight vs dist | contains check in Set |
| **Duplicates in pq** | allowed, skipped later | never processed twice |
| **Code complexity** | simpler ✅ | slightly more code |
| **Performance** | same O((V+E) log V) | same O((V+E) log V) |
 
> Both approaches are correct! `d > dist[v]` is cleaner and more commonly used in competitive programming.
 
---
