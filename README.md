# Hybrid Algorithm for Global Shortest Path

The code implements a hybrid algorithm to find the the shortest path between *any* two nodes in a weighted, directed graph.

The algorithm adapts its strategy based on graph size to balance accuracy and performance.

## Phase 1: Negative Cycle Detection (Bellman-Ford)

First, the algorithm checks for negative-weight cycles.

1.  A "super-source" node is added with a 0-weight edge to all other nodes.
2.  **Bellman-Ford** is run from this super-source.
3.  If a negative cycle is detected, the path is undefined. The function stops and returns `float('-inf')`.

## Phase 2: Hybrid Strategy

If no negative cycles exist, the algorithm chooses a strategy based on a size threshold ($V=150$ nodes).

---

### Case A: Small Graphs ($V \le 150$) — Exact Solution

For small graphs, the algorithm finds the precise solution using **Johnson's Algorithm**.

1.  **Reweighting**: It uses the Bellman-Ford results from Phase 1 to reweight all graph edges to be non-negative, preserving the shortest paths.
2.  **Dijkstra $V$-times**: It runs **Dijkstra's** algorithm $V$ times (once from each node) on the safe, reweighted graph.
3.  **Global Minimum**: It scans all $V \times (V-1)$ paths and returns the one with the absolute lowest cost.

**Result**: This method **guarantees** finding the true global shortest path.

---

### Case B: Large Graphs ($V > 150$) — Landmark Heuristic

For large graphs, Johnson's algorithm is too slow. The algorithm switches to a faster **approximation heuristic**.

1.  **Landmark Selection**: A small subset of $k$ "strategic" nodes (e.g., $k=30$) is selected.
2.  **Dijkstra $k$-times**: **Dijkstra's** algorithm is run $k$ times, starting from each of the $k$ landmarks.
3.  **Partial Minimum**: The function returns the shortest path found *originating* from one of these landmarks.

**Result**: This method is an **approximation**. It is much faster but **not guaranteed** to find the global minimum, as the true path might be between two non-landmark nodes.