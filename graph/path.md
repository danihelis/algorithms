---
order: 4
---

# Shortest Paths

Given a graph $G=(V, E)$ and a distance function $d:E \rightarrow \mathbb{N}$,
we want to find the shortest path between two vertices $u, v \in V$.  The
shortest path $p \in E^*$ is a sequence of edges such that $u
\overset{p}{\leadsto} v$ and $\sum_{e \in p}{d(e)}$ is minimal.


The algorithms below use [adjacency lists](./representation.md#adjacency-list) to
represent the graph $G=(V, E)$. Each vertex $v \in V$ contains a list of edges
$E(v) \subseteq E$ that connect it to another vertex $u \in V$. We will denote
this set of connected vertices as $V(v) \subseteq V$.

```c
#define MAX_VERTICES  1000

typedef struct {
    int edge[MAX_VERTICES][MAX_VERTICES];
    int distance[MAX_VERTICES][MAX_VERTICES];  /* edge's distance */
    int num_edges[MAX_VERTICES];
    int num_vertices;
} graph_t;

```

## For non-negative distances

Find the shortest path between two vertices of a graph where the distances
between two vertices are non-negative.

**Input** A graph $G=(V,E)$ with distance function $d:E \rightarrow \mathbb{N}$,
and two vertices $u, v \in V$ \
**Output** The shortest path from $u$ to $v$ \
**Time** $O(|V|^2)$

This algorithm is attributed to Dijkstra. Initially, we set the shortest
distance $\delta$ from $u$ to all other vertices as infinite. We then iterate
over the non visited vertices, starting with $u$. Let the current node be $v$.
We mark $v$ as visited and update the shortest distance of all its connected
vertices $x \in V(v)$, setting $\delta(x)$ as $\delta(v) + d(v, x)$ if the new
distance is shorter.  We repeat this process by selecting the next non visited
vertex with shortest distance $\delta$.  Eventually we will reach the
destination vertex (if it is connected), and we can reconstruct the shortest
path using the edges we select during the distance updates.

```c
#define INFINITY  (MAXINT >> 1)

typedef struct {
    ...
    char visited[MAX_VERTICES];  /* either 0 or 1 */
    int shortest[MAX_VERTICES];
    int prior[MAX_VERTICES]; /* connecting vertex in shortest path */
} graph_t;

int shortest_path(graph_t *graph, int source, int destiny) {
    int v, e, current;
    for (v = 0; v < graph->num_vertices; v++) {
        graph->visited[v] = 0;
        graph->shortest[v] = v == source ? 0 : INFINITY;
    }
    current = source;
    while (current != destiny && current != -1) {
        graph->visited[current] = 1;
        if (current == destiny) break;
        for (e = 0; e < graph->num_edges[current]; e++) {
            int next = graph->edge[e];
            if (graph->visited[next]) continue;
            int new_distance = graph->shortest[current]
                    + graph->distance[current][e];
            if (new_distance < graph->shortest[next]) {
                graph->shortest[next] = new_distance;
                graph->prior[next] = current;
            }
        }
        for (current = -1, v = 0; v < graph->vertices; v++) {
            if (!graph->visited[v] && (current == -1
                    || graph->shortest[i] < graph->shortest[current])) {
                current = v;
            }
        }
    }
    if (current == -1) return INFINITY;
    return graph->shortest[destiny];
}
```

We can easily extend this algorithm to compute the shortest path from $u$ to all
other vertices. We simply repeat the `while` loop until all reachable edges have
been visited. The algorithm still runs in $O(|V|^2)$.


## For negative distances

Find the shortest path between two vertices of a graph where the distances
between two vertices may be negative.

**Input** A graph $G=(V,E)$ with distance function $d: E \rightarrow \mathbb{Z}$,
 and two vertices $u, v \in V$ \
**Output** The shortest path from $u$ to any other vertex \
**Time** $O(|V| \cdot |E|)$

Dijkstra's algorithm does not work when distances are negative, so the problem
requires a different approach. The algorithm below is attributed to Bellman &
Ford. We start by setting the shortest distances $\delta$ from $u$ to all other
vertices as infinite. Then, for each edge $(i, j) \in E$, we relax the shortest
distance $\delta(j)$ by setting it as $\delta(i) + d(i, j)$, if it is shorter.
We apply this relaxation $|V| - 1$ times, which will compute the
shortest path from $u$ to any other vertex. However, if there is a cycle in the
graph with negative distance, the shortest path cannot be computed, since we
can loop indefinitely in the cycle to reduce the distance.

To detect if such a cycle exists, we execute one more relaxation on the shortest
distances. If it would reduce any distance even further, then a negative cycle
must exist. Bellman-Ford's algorithm is frequently used to detect this type of
cycle.

```c
#define INFINITY    (MAXINT >> 1)

typedef struct {
    ...
    int shortest[MAX_VERTICES];
    int prior[MAX_VERTICES]; /* connecting vertex in shortest path */
} graph_t;

int shortest_path(graph_t *graph, int source, int destiny) {
    int i, j, k, e, new_distance;
    for (i = 0; i < graph->num_vertices; i++) {
        graph->shortest[i] = i == source ? 0 : INFINITY;
    }
    for (k = 0; k < graph->num_vertices; k++) {
        for (i = 0; i < graph->num_vertices; i++) {
            for (e = 0; e < graph->num_edges[i]; e++) {
                j = graph->edge[i][e];
                new_distance = graph->shortest[i] + graph->distance[i][e];
                if (new_distance < graph->shortest[j]) {
                    /* The last loop detects negative cycle */
                    if (k == graph->num_vertices - 1) return -INFINITY;
                    graph->shortest[j] = new_distance;
                    graph->prior[j] = i;
                }
            }
        }
    }
    return graph->shortest[destiny];
}
```

## For all paths

Find the distance of the shortest path between all vertices. Distances can be
negative, but the graph has no negative cycle.

**Input** A graph $G=(V,E)$ with distance function $
d:E \rightarrow \mathbb{Z}$ \
**Output** The distance of the shortest path between all vertices \
**Time** $O(|V|^3)$

This algorithm is attributed to Floyd & Marshall. We use a strategy similar to
Bellman-Ford's. Let $\delta:V^2 \rightarrow \mathbb{Z}$ be the shortest distance
between two vertices. Initially, $\delta(i, j)$ equals to $d(i,j)$ if vertices
$i, j \in V$ are connected, or infinite otherwise. Then, for each vertex $k \in
V$, we relax the distance for each pair $i, j \in V$ using the distance
$\delta(i, k) + \delta(k, j)$ if it is shorter than $\delta(i, j)$.  It will
eventually produce the shortest distances for each pair.

Notice that if the distances were non-negative we could run Dijkstra's to obtain
all shortest paths, executing it $|V|$ times. It would have the same time
complexity $O(|V|^3)$.

```c
#define INFINITY    (MAXINT >> 1)

typedef struct {
    ...
    int shortest[MAX_VERTICES][MAX_VERTICES];
} graph_t;

void all_shortest_paths(graph_t *graph) {
    int k, i, j, e;
    for (i = 0; i < graph->num_vertices; i++) {
        for (j = 0; j < graph->num_vertices; j++) {
            graph->shortest[i][j] = i == j ? 0 : INFINITY;
        }
        for (e = 0; e < graph->num_edges[i]; e++) {
            graph->shortest[i][graph->edge[e]] = graph->distance[i][e];
        }
    }
    for (k = 0; k < graph->num_vertices; k++) {
        for (i = 0; i < graph->num_vertices; i++) {
            for (j = 0; j < graph->num_vertices; j++) {
                int new_distance = graph->shortest[i][k] +
                        graph->shortest[k][j];
                if (graph->shortest[i][j] > new_distance) {
                    graph->shortest[i][j] = new_distance;
                }
            }
        }
    }
}
```
