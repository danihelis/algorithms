---
order: 5
---

# Minimal Spanning Trees

Consider a connected undirected graph $G=(V,E)$ with a distance function $d: E
\rightarrow \mathbb{Z}$.  A spanning tree for $G$ is a subgraph spanning over
all its vertices such that, for each pair of vertices $u, v \in V$, there is a
single path $p \in E^*$ that connects them, $u \overset{p}{\leadsto} v$. There
are exactly $|V| - 1$ edges in the tree, since all vertices are connected.
A minimum spanning tree is a spanning tree in which the sum of edge distances
$\sum_{e\in E} d(e)$ is minimal.

There are two different strategies to compute a minimal spanning tree for a
graph, detailed below. We will use
[adjacency lists](./representation.md#adjacency-list)
to represent a graph, and an explicit list of edges to represent the minimal
spanning tree.

```c
#define MAX_VERTICES  1000
#define MAX_EDGES     (MAX_VERTICES)  /* edges per vertex */

typedef struct {
    int edge[MAX_VERTICES][MAX_EDGES];
    int distance[MAX_VERTICES][MAX_EDGES];
    int num_edges[MAX_VERTICES];
    int num_vertices;
} graph_t;

typedef struct {
    int u, v, distance;
} edge_t;

edge_t tree[MAX_VERTICES];
```

## Using disjoint sets

Given a connected undirected graph and a distance function, compute a minimal
spanning tree that spans over its vertices. This strategy uses
[union-find disjoint sets](../structure/set.md) and is attributed to Kruskal.

**Input** A graph $G=(V,E)$ and a distance function $d: E
\rightarrow \mathbb{Z}$ \
**Output** A minimal spanning tree over $G$ \
**Time** $O(|E| \log{|V|})$

Kruskal's algorithm is quite simple. Initially, the tree is empty, and each
vertex is part of a unique disconnected component. We use
[disjoint sets](../structure/set.md) to represent these components. Then, we
sort all edges $E$ in increasing distance order, and iterate over each edge
until all components are connected.  If an edge links together two vertices that
are not connected yet, we add the edge to the tree and connect the vertices by
joining their sets.

The code below uses [`qsort`](http://www.cplusplus.com/reference/cstdlib/qsort/)
to sort the edges in $O(n \log{n})$, but any
[sorting algorithm](../structure/vector.md#merge-sort) with the same time
complexity will do.

```c
typedef struct {
    ...
    int set[MAX_VERTICES];
} graph_t;

int compare_edges(const void *e1, const void *e2) {
    return ((edge_t *) e1)->distance - ((edge_t *) e2)->distance;
}

int find_set(graph_t *graph, int vertex) {
    if (graph->set[vertex] == vertex) return vertex;
    graph->[vertex] = find_set(graph->set[vertex]);
    return graph->set[vertex];
}

int set_union(graph_t *graph, int a, int b) {
    a = find_set(graph, a);
    graph->set[a] = find_set(graph, b);
    return graph->set[a];
}

int mst(graph_t *graph, edge_t tree[]) {
    int u, v, e, num_edges, tree_size, total_distance;
    edge_t edge_list[MAX_VERTICES * MAX_EDGES];
    num_edges = 0;
    for (v = 0; v < graph->num_vertices; v++) {
        graph->set[v] = v;
        for (e = 0; e < graph->num_edges[v]; e++, num_edges++) {
            edge_list[num_edges].v = v;
            edge_list[num_edges].u = graph->edge[v][e];
            edge_list[num_edges].distance = graph->distance[v][e];
        }
    }
    qsort(edge_list, num_edges, sizeof (edge_t), compare_edges);
    for (e = tree_size = total_distance = 0;
            tree_size < graph->num_vertices - 1 && e < num_edges; e++) {
        edge_t *edge = &edge_list[e];
        u = find_set(edge->u);
        v = find_set(edge->v);
        if (u != v) {
            set_union(u, v);
            tree[tree_size++] = *edge;
            total_distance = edge->distance;
        }
    }
    return total_distance;
}
```

## Using breadth-first search

Given a connected undirected graph and a non-negative distance function, compute
a minimal spanning tree that spans over its vertices. This strategy uses
[breadth-first search](./traversal.md#breadth-first-search) in a similar way to
[Dijkstra's shortest path](./path.md#for-non-negative-distances) and is
attributed to Prim.

**Input** A graph $G=(V,E)$ and a distance function $d: E \rightarrow
\mathbb{N}$ \
**Output** A minimal spanning tree over $G$ \
**Time** $O(|V|^2)$

Prim's algorithm is almost identical to
[Dijkstra's](./path.md#for-non-negative-distances), and is also restricted to
non-negative distance values. We compute the shortest path from the first vertex
(chosen arbitrarily) to all other vertices $v \in V$, along with its distance
$\delta(v)$. After the breadth-first search is complete, we select the edges
that compose the shortest path from the first vertex to all other vertices. We
can easily recover the distance of an edge $d(u, v)$ by computing the difference
between the shortest distances of its vertices, $\delta(v) - \delta(u)$.

```c
#define INFINITY  (MAXINT >> 1)

typedef struct {
    ...
    char visited[MAX_VERTICES];  /* either 0 or 1 */
    int shortest[MAX_VERTICES];
    int prior[MAX_VERTICES];  /* connecting vertex in shortest path */
} graph_t;

int mst(graph_t *graph, edge_t tree[]) {
    int v, e, current, total_distance;
    for (v = 0; v < graph->num_vertices; v++) {
        graph->visited[v] = 0;
        graph->shortest[v] = v == 0 ? 0 : INFINITY;
    }
    current = 0;
    while (current != -1) {
        graph->visited[current] = 1;
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
    for (v = 1, total_distance = 0; v < graph->num_vertices; v++) {
        edge_t edge;
        edge.v = v;
        edge.u = graph->prior[v];
        edge.distance = graph->shortest[v] - graph->shortest[edge.u];
        total_distance += edge.distance;
        tree[v - 1] = edge;
    }
    return total_distance;
}
```

Prim's algorithm outperforms Kruskal's when dealing with dense graphs, where
$|E| = O(|V|^2)$. Otherwise, Kruskal's is faster.
