# Hybrid Algorithm for Global Shortest **Positive** Path

The code implements a hybrid algorithm to find the shortest **strictly positive** path between *any* two nodes in a weighted, directed graph.

Edge weights can be positive, zero, or negative. The algorithm:

- Detects and reports negative-weight cycles,
- Allows negative edges,
- But, in the final result, returns only the **shortest path with total cost > 0**.

If no such path exists, it signals that explicitly (see **Return Values** below).

The algorithm adapts its strategy based on graph size to balance accuracy and performance.

---

## Phase 1: Negative Cycle Detection (Bellman–Ford)

First, the algorithm checks for negative-weight cycles.

1. A "super-source" node is added with a 0-weight edge to all other nodes.  
2. **Bellman–Ford** is run from this super-source.  
3. If a negative cycle is detected, the shortest positive path is undefined: any path cost can be made arbitrarily small.

In this case, the function stops and returns:

```python
float('-inf')
```

to indicate the presence of a negative cycle.

---

## Phase 2: Hybrid Strategy (No Negative Cycles)

If no negative cycles exist, the algorithm chooses a strategy based on a size threshold ($V = 150$ nodes).

The internal shortest-path computations can still produce **negative** distances (because negative edges are allowed), but the *final selection step* will consider **only paths with total cost > 0** and pick the smallest among them.

---

### Case A: Small Graphs ($V \le 150$) — Exact All-Pairs + Positive Filter

For small graphs, the algorithm computes all-pairs shortest paths exactly using **Johnson’s Algorithm**.

1. **Reweighting**: The algorithm uses a Bellman–Ford-based reweighting (as in Johnson’s algorithm) so that all edges in the transformed graph are non-negative, while preserving shortest paths.  
2. **Dijkstra $V$-times**: It runs **Dijkstra’s** algorithm $V$ times (once from each node) on the reweighted graph.  
3. **Global Positive Minimum**: From the resulting all-pairs distance matrix, it scans all $V 	imes (V-1)$ pairs and:
   - **Ignores** all paths whose total cost is $\le 0$  
   - Among the remaining ones, returns the path with the **smallest strictly positive** cost.

**Result**:  
- This method **guarantees** finding the true global shortest path among all paths with total cost > 0.  
- Negative and zero-cost paths may exist and are correctly computed internally, but they are **not** selected as the final answer.

---

### Case B: Large Graphs ($V > 150$) — Landmark Heuristic + Positive Filter

For large graphs, Johnson’s algorithm is too slow. The algorithm switches to a faster **approximation heuristic** based on **landmarks**.

1. **Landmark Selection**: A small subset of $k$ "strategic" nodes (e.g., $k = 30$) is selected as landmarks.  
2. **Dijkstra $k$-times**: **Dijkstra’s** algorithm is run $k$ times, each from a different landmark.  
3. **Partial Positive Minimum**: Among all paths discovered **originating from landmarks**, the algorithm:
   - **Ignores** paths whose total cost is $\le 0$  
   - Returns the path with the smallest strictly positive cost.

**Result**:  
- This method is an **approximation**: it only explores paths starting from the chosen landmarks.  
- It is much faster but **does not guarantee** finding the global shortest positive path, because the optimal positive path might be between two non-landmark nodes.

---

## Return Values

Let:

```python
dist, info = shortest_path(problem)
```

- **Negative cycle detected**:  
  - `dist == float('-inf')`  
  - `info is None`  
  - Meaning: A negative-weight cycle exists; shortest positive path is undefined.

- **At least one strictly positive path found**:  
  - `dist > 0` (finite)  
  - `info` typically contains details such as `(source, target, path_nodes)`  
  - Meaning: `dist` is the smallest total cost among all paths with cost > 0.

- **No strictly positive path exists (but no negative cycle)**:  
  - `dist == float('inf')`  
  - `info is None`  
  - Meaning: Every reachable path has total cost $\le 0`, so there is **no** path with strictly positive total weight.

---

In summary:

- The algorithm still works with graphs having negative edges.  
- It still detects negative cycles and reports them with `-inf`.  
- But the final answer is always the **shortest strictly positive path**, or an explicit signal that no such path exists.

