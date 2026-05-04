# UNIT 2 — Graph Algorithms: Shortest Paths

## 2.1 Graph Fundamentals (Quick Reference)

| Term                | Definition                                          |
| ------------------- | --------------------------------------------------- |
| **Graph G**         | G = (V, E) — a set of vertices and edges            |
| **Directed Graph**  | Edges have direction (u to v)                       |
| **Weighted Graph**  | Edges carry numeric weights                         |
| **Negative Weight** | An edge with weight less than 0                     |
| **Negative Cycle**  | A cycle whose total edge weight is less than 0      |
| **Relaxation**      | Update dist[v] if a shorter path through u is found |
| **Sparse Graph**    | Number of edges E is approximately O(V)             |
| **Dense Graph**     | Number of edges E is approximately O(V^2)           |

### Relaxation — Core Operation

```cpp
if (dist[u] + w(u, v) < dist[v]) {
    dist[v] = dist[u] + w(u, v);
}
```

## 2.2 Dijkstra's Algorithm (Prerequisite / Reference)

> Single-source shortest path for non-negative weights only.

- Uses a min-heap (priority queue)
- Time: **O((V + E) log V)**
- Cannot handle negative edge weights

```cpp
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
dist[src] = 0;
pq.push({0, src});

while (!pq.empty()) {
    auto [d, u] = pq.top(); pq.pop();
    if (d > dist[u]) continue;
    for (auto [v, w] : adj[u]) {
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            pq.push({dist[v], v});
        }
    }
}
```

---

## 2.3 Bellman-Ford Algorithm

### Definition

Single-source shortest path algorithm that handles **negative edge weights** and **detects negative cycles**.

### Core Idea

Relax all edges exactly **V − 1** times. After V − 1 iterations, all shortest paths are finalized, since a shortest path visits at most V − 1 edges in a graph with no negative cycles.

### Algorithm

```
Initialize:
    dist[source] = 0
    dist[all others] = infinity

Repeat V-1 times:
    For each edge (u, v, w):
        if dist[u] + w < dist[v]:
            dist[v] = dist[u] + w

Negative cycle check (one additional pass):
    For each edge (u, v, w):
        if dist[u] + w < dist[v]:
            -> Negative cycle detected
```

```cpp
vector<tuple<int,int,int>> edges; // {u, v, w}
vector<int> dist(V, INT_MAX);
dist[src] = 0;

for (int i = 0; i < V - 1; i++) {
    for (auto [u, v, w] : edges) {
        if (dist[u] != INT_MAX && dist[u] + w < dist[v])
            dist[v] = dist[u] + w;
    }
}

// Negative cycle detection
for (auto [u, v, w] : edges) {
    if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {
        cout << "Negative cycle exists!" << endl;
    }
}
```

### Worked Example

```
Edges: A->B (4), A->C (5), B->C (-3)

After iteration 1:
  dist[A] = 0, dist[B] = 4, dist[C] = 1  (via A->B->C = 4 + (-3) = 1)
```

### Complexity

| Metric | Value    |
| ------ | -------- |
| Time   | O(V x E) |
| Space  | O(V)     |

### Advantages vs Limitations

| Advantages               | Limitations                         |
| ------------------------ | ----------------------------------- |
| Handles negative weights | Slower than Dijkstra                |
| Detects negative cycles  | Inefficient for dense graphs        |
| Simple implementation    | Not optimal for large sparse graphs |

---

## 2.4 Floyd-Warshall Algorithm

### Definition

An **all-pairs shortest path** algorithm using dynamic programming. Finds shortest paths between every pair of vertices.

### Core Idea

For each intermediate vertex k, check whether routing through k improves the path from i to j.

### Recurrence

```
dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
```

### Algorithm

```cpp
// Initialize: dist[i][j] = edge weight, or INF if no edge, 0 if i == j
for (int k = 0; k < V; k++)
    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            if (dist[i][k] != INF && dist[k][j] != INF)
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);

// Negative cycle detection
for (int i = 0; i < V; i++)
    if (dist[i][i] < 0) // negative cycle exists
```

### Worked Example

```
Initial:
       A    B    C
  A  [ 0    3    INF ]
  B  [ INF  0    1   ]
  C  [ 2    INF  0   ]

After all iterations:
       A    B    C
  A  [ 0    3    4   ]   <- A->B->C = 3 + 1 = 4
  B  [ 3    0    1   ]   <- B->C->A = 1 + 2 = 3
  C  [ 2    5    0   ]   <- C->A->B = 2 + 3 = 5
```

### Path Reconstruction

Use a `next[i][j]` matrix to record intermediate vertices.

```cpp
int next[V][V]; // next[i][j] = next vertex on path from i to j

// Reconstruct path from i to j:
while (i != j) {
    path.push_back(i);
    i = next[i][j];
}
path.push_back(j);
```

### Complexity

| Metric | Value  |
| ------ | ------ |
| Time   | O(V^3) |
| Space  | O(V^2) |

### Advantages vs Limitations

| Advantages                        | Limitations                           |
| --------------------------------- | ------------------------------------- |
| Finds all-pairs shortest paths    | O(V^3) is too slow for large graphs   |
| Simple triple-loop implementation | Requires O(V^2) memory                |
| Handles negative weights          | Does not scale to large sparse graphs |
| Easy negative cycle detection     |                                       |

## 2.5 Johnson's Algorithm

### Definition

An **all-pairs shortest path** algorithm designed for sparse graphs. Combines **Bellman-Ford** (for reweighting) with **Dijkstra** (run from every vertex).

### Why Reweighting?

Dijkstra is fast but cannot handle negative weights. Johnson's transforms all edge weights to be non-negative without changing the relative shortest paths.

### Algorithm Steps

```
1. Add a new vertex S, connected to all vertices with weight 0.

2. Run Bellman-Ford from S to compute h[v] for all v:
       h[v] = shortest distance from S to v

3. Reweight each edge (u, v, w):
       w'(u, v) = w(u, v) + h[u] - h[v]
   (All reweighted edges are guaranteed >= 0 by triangle inequality)

4. Run Dijkstra from every vertex using the reweighted graph.

5. Recover true distances:
       dist(u, v) = dijkstra_dist(u, v) - h[u] + h[v]
```

```cpp
// Reweighting step
for (auto& [u, v, w] : edges)
    w = w + h[u] - h[v]; // w' is always >= 0
```

### Complexity

| Step               | Complexity      |
| ------------------ | --------------- |
| Bellman-Ford       | O(VE)           |
| Dijkstra x V times | O(V \* E log V) |
| Total              | O(VE log V)     |
| Space              | O(V^2)          |

### Advantages vs Limitations

| Advantages                                  | Limitations                     |
| ------------------------------------------- | ------------------------------- |
| Efficient for sparse graphs                 | Cannot handle negative cycles   |
| Handles negative weights                    | More complex to implement       |
| Better than Floyd-Warshall on sparse graphs | Overhead from Bellman-Ford step |

## 2.6 Negative Cycles

### Definition

A **negative cycle** is a cycle in a graph where the sum of all edge weights is less than zero.

```
Formal: sum of w(vi, vi+1) < 0
```

### Example

```
A -> B  (1)
B -> C  (-2)
C -> A  (-2)

Total = 1 + (-2) + (-2) = -3  <- Negative Cycle
```

### Implications

- The shortest path becomes undefined — it can always be reduced further by traversing the cycle again.
- Any algorithm assuming finite shortest paths will break down.
- Must be detected and reported rather than resolved.

### Detection Methods

| Algorithm          | Detection Method                                                    |
| ------------------ | ------------------------------------------------------------------- |
| **Bellman-Ford**   | If relaxation is still possible after V-1 iterations                |
| **Floyd-Warshall** | If `dist[i][i] < 0` for any vertex i                                |
| **Johnson's**      | Cannot detect directly — Bellman-Ford step will catch it beforehand |

---

## 2.7 Algorithm Comparison Table

| Feature                      | Dijkstra                | Bellman-Ford     | Floyd-Warshall        | Johnson's                 |
| ---------------------------- | ----------------------- | ---------------- | --------------------- | ------------------------- |
| **Type**                     | Single-source           | Single-source    | All-pairs             | All-pairs                 |
| **Negative Weights**         | No                      | Yes              | Yes                   | Yes                       |
| **Negative Cycle Detection** | No                      | Yes              | Yes                   | No                        |
| **Time Complexity**          | O((V+E) log V)          | O(VE)            | O(V^3)                | O(VE log V)               |
| **Space Complexity**         | O(V)                    | O(V)             | O(V^2)                | O(V^2)                    |
| **Best For**                 | Non-neg weights, sparse | Negative weights | Small or dense graphs | Sparse + negative weights |

## 2.8 When to Use Which Algorithm

```
Single-source shortest path?
    No negative weights          -> Dijkstra (fastest)
    Negative weights possible    -> Bellman-Ford
    Need to detect neg. cycle    -> Bellman-Ford

All-pairs shortest path?
    Small or dense graph (V <= ~400)  -> Floyd-Warshall (simplest)
    Large sparse graph                -> Johnson's (most efficient)
```

## 2.9 Time Complexity Summary

| Algorithm      | Time           | Space  |
| -------------- | -------------- | ------ |
| Dijkstra       | O((V+E) log V) | O(V)   |
| Bellman-Ford   | O(V x E)       | O(V)   |
| Floyd-Warshall | O(V^3)         | O(V^2) |
| Johnson's      | O(VE log V)    | O(V^2) |
