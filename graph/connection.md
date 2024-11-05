---
order: 3
---

# Connections

In a graph $G=(V,E)$, two vertices $u,v \in V$ are connected if $(u,v) \in E$,
or there is $k \in V, (u,k) \in E$ such that $k, v$ are connected. The sequence
of edges that connects two vertices is called a path $p \in E^*$. We will denote
this connection using the notation $u \overset{p}{\leadsto} v$, with $u, v \in
V$. In a undirected graph, if $u \overset{p}{\leadsto} v$, then $v
\overset{p}{\leadsto} u$.

If a vertex $v \in V$ is connected to itself by a non empty path $p \in E^*$,
that is, $v \overset{p}{\leadsto} v$, then $p$ is called a *cycle* and the graph
is called *cyclic*. A directed acyclic graph is abbreviated DAG.

In graph $G=(V,E)$, a *connected component* is a maximal subset
$C(v) \subseteq V$ that contains $v \in V$ and all vertices $u \in V$ such that
$v \leadsto u$. It follows that two connected components have no path connecting
their vertices. These components are *disconnected* in relation to each other.

In a directed graph, a *strongly connected component* is a maximal subset $C(v)
\subseteq V$ that contains $v \in V$ and all vertices that are part of a cycle
containing $v$, that is, $\forall u \in V: v \leadsto u \leadsto v$. Two
strongly connected components can have a connection between their vertices, but
it cannot form a cycle. A DAG has no strongly connected components.

In the following algorithms, we will use [adjacency lists](
./representation.md#adjacency-lists) to represent a graph.

```c
#define MAX_VERTICES  1000

typedef struct {
    int edge[MAX_VERTICES][MAX_VERTICES];
    int num_edges[MAX_VERTICES];
    int num_vertices[MAX_VERTICES];
    int component[MAX_VERTICES]; /* index from 0 to num_components-1 */
    int num_components;
} graph_t;
```

## Connected Components

Given an undirected graph $G=(V, E)$, compute the number of maximal connected
components in $G$. A vertex $u \in V$ is part of a connected component $C(v)
\subseteq V, v \in V$, if and only if $u \leadsto v$.

**Input** An undirected graph $G=(V,E)$ \
**Output** A list of all connected components $\{C_1, \ldots, C_n\}$ of $G$ \
**Time** $O(|E| + |V|)$

We simply run a [depth-first search](./traversal.md#depth-first-search)
for each non-visited vertex of the graph.
All vertices reached from a non-visited vertex $v$ are part of the component
that contains $v$.

```c
#define <string.h>

typedef struct {
    ...
    char visited[MAX_VERTICES]; /* either 0 or 1 */
} graph_t;

void visit_vertex(graph *graph, int vertex) {
    int e;
    graph->visited[vertex] = graph->num_components;
    graph->component[vertex] = component;
    for (e = 0; e < graph->num_edges[vertex]; e++) {
        int next = graph->edge[vertex][e];
        if (!graph->visited[next]) visit_vertex(graph, next);
    }
}

void get_connected_components(graph_t *graph) {
    int v;
    memset(graph->visited, 0, sizeof (int) * graph->num_vertices);
    graph->num_components = 0;
    for (v = 0; v < graph->num_vertices; v++) {
        if (!graph->visited[i]) visit_vertex(graph, v);
        graph->num_components++;
    }
}
```

## Strongly Connected Components

Given a directed graph $G=(V, E)$, compute the strongly connected components of
$G$.  A vertex $u \in V$ is part of a strongly connected component $C(v)
\subseteq V, v \in V$, if and only if $u \leadsto v \leadsto u$.

**Input** A directed graph $G=(V,E)$ \
**Output** A list of all strongly connected components $\{C_1, \ldots, C_n\}$
of $G$ \
**Time** $O(|E| + |V|)$

We use an adapted version of [depth-first
search](./traversal.md#depth-first-search) to traverse the graph and annotate
the strongly connected components. The idea is similar to the previous approach:
we start from a particular vertex $v$ and explore its connected vertices. If we
find a back edge (an edge that connects an already visited vertex), we
must be in a cycle, and these vertices must be part of a strongly connected
component.

To detect a back edge, we use two timestamps while traversing the graph.  The
first, `visited`, marks the first time we visit a vertex. The second, `low`,
indicates the lowest timestamp that can be reached from the current vertex.
This value equals `visited` (its own timestamp) if there are no back edges,
otherwise it will have the lower value of a vertex we visited earlier.

While we traverse the graph, we push the visited vertices in a special stack.
If the vertex finds a back edge, we simply continue execution. When we finish
the exploration of a vertex with no back edges, all possible connected vertices
have been explored and processed, except the ones with back edges. It must
follow that these vertices are part of the same component. We remove them all
from the stack, annotate the component, and resume the exploration of a previous
vertex. Note that if previous vertices connect the current one, they are not
part of the component, since the current vertex does not have a lower value
pointing to them.

(We use another flag, `processed`, to differentiate between two executions of
`visit_vertex`, in case the second execution finds vertices that connect the
ones from the first execution. These connections should not be considered back
edges.)

```c
#define <string.h>

typedef struct {
    ...
    int visited[MAX_VERTICES];  /* timestamp of visit or 0 */
    int low[MAX_VERTICES];  /* lowest reachable timestamp */
    char processed;  /* either 0 or 1 */
} graph_t;

void visit_vertex(graph_t *graph, int vertex, int *counter, int stack[],
                  int *top) {
    int e;
    (*counter)++;
    graph->low[vertex] = graph->visited[vertex] = *counter;
    stack[(*top)++] = vertex;
    for (e = 0; e < graph->num_edges[vertex]; e++) {
        int next = graph->edge[vertex][e];
        if (!graph->visited[next]) {  /* recursive exploration */
            visit_vertex(graph, next, counter, stack, top);
        }
        if (!graph->processed[next] && graph->low[next] < graph->low[vertex]) {
            graph->low[vertex] = graph->low[next];  /* back edge */
        }
    }

    /* No back edge; reached vertices form maximal component */
    if (graph->low[vertex] == graph->visited[vertex]) {
        int v = -1;
        while (*top > 0 && v != vertex) {
            v = stack[--(*top)];
            graph->component[v] = graph->num_components;
            graph->processed[v] = 1;
        }
        graph->num_components++;
    }
}

void get_strong_components(graph_t *graph) {
    int stack[MAX_VERTICES], top, v, counter;
    for (v = 0; v < graph->num_vertices; v++) {
        graph->visited[v] = graph->processed[v] = 0;
    }
    top = 0, counter = 0, graph->num_components = 0;
    for (v = 0; v < graph->num_vertices; v++) {
        if (!graph->visited[v]) {
            visit_vertex(graph, v, &counter, stack, &top);
        }
    }
}
```


## Articulation Points and Bridges

Given a connected component from a graph, we define an *articulation point* as a
vertex that, once removed, will cause the component to become disconnected.
Likewise, we define a *bridge* as an edge that, once removed, will cause the
component to become disconnected.

Given a graph, we want to find a list of articulation points and bridges for
each of its connected components.

**Input** A graph $G=(V,E)$ \
**Output** A list of articulation points $A \subseteq V$, and
        bridges $B \subseteq E$ \
**Time** $O(|E| + |V|)$

We use a strategy similar to the previous one used to detect [strongly connected
components](#strongly-connected-components).  We execute a [depth-first
search](./traversal.md#depth-first-search), and annotate a timestamp (`visited`)
that indicates the first time we visit a vertex, and the lowest timestamp
(`low`) that can be reached by this vertex. These values are used to detect back
edges.

To determine the articulation points in a graph,
there are two cases to consider:

1. If the vertex is the first one visited, then it is an articulation point if
   and only if it connects two vertices that are not otherwise connected.

2. If the vertex is not the first, then it is an articulation point if and only
   if one of its connected vertices has no back edge to a vertex that was
   previously visited. Using the timestamps defined above, we have that a vertex
   is an articulation point if and only if one of its connected vertices has a
   `low` value equal to or larger than the vertex's `low`.

Likewise, an edge will be a bridge if and only if the connected vertex have no
back edge to a previously visited vertex, that is, if the connected vertex has a
`low` value strictly larger than the vertex's `low`. (If it is equal, it means
that there are other edges that connect both vertices.)

```c
#define <string.h>

typedef struct {
    ...
    int visited[MAX_VERTICES];  /* timestamp of visit or 0 */
    int low[MAX_VERTICES];  /* lowest reachable timestamp */
    char processed;  /* either 0 or 1 */
    char articulation[MAX_VERTICES]; /* either 0 or 1 */
    char bridge[MAX_VERTICES][MAX_VERTICES]; /* either 0 or 1 */
} graph_t;


void visit_vertex(graph_t *graph, int vertex, int *counter, char first) {
    int e, connections;
    (*counter)++;
    graph->low[vertex] = graph->visited[vertex] = *counter;
    different = 0;
    for (e = 0; e < graph->num_edges[vertex]; e++) {
        int next = graph->edge[vertex][e];
        if (!graph->visited[next]) {  /* recursive exploration */
            visit_vertex(graph, next, counter, 0 /* not first */);
            graph->articulation[vertex] |= (graph->low[next] >=
                    graph->low[vertex]);
            graph->bridge[vertex][next] |= (graph->low[next] >
                    graph->low[vertex]);
            connections++;
        }
        if (!graph->processed[next] && graph->low[next] < graph->low[vertex]) {
            graph->low[vertex] = graph->low[next];  /* back edge */
        }
    }
    /* Particular case: first visited */
    if (first) graph->articulation[vertex] = connections > 1;
}

void get_articulations_and_bridges(graph_t *graph) {
    int v, e, counter;
    for (v = 0; v < graph->num_vertices; v++) {
        graph->visited[v] = graph->processed[v] = graph->articulation[v] = 0;
        for (e = 0; e < graph->num_edges[e]; e++) {
            graph->bridge[v][e] = 0;
        }
    }
    for (v = 0; v < graph->num_vertices; v++) {
        if (!graph->visited[v]) {
            visit_vertex(graph, v, &counter, 1 /* first */);
        }
    }
}
```

----------

# Draft

## Flood Fill

**Input** A graph $G=(V,E)$ with mapping $m:V\rightarrow \mathbb{N}$,
        an element $v \in V$ and a value $x \in \mathbb{N}$ \
**Output** A new mapping $m$ such that
         $m'(u) = x, \forall u \in C_v: m(u) = m(v)$, where $C_v \subseteq V$
         is a connected component that contains $v$ \
**Time** $O(|V| + |E|)$

```c
    void flood_fill(graph_t *graph, int map[], int v, int x) {
        int component[MAX_VERTICES];
        int i, n, old_value;

        get_components(graph, component, &n);
        old_value = map[v];
        for (i = 0; i < graph->vertices; i++)
            if (component[i] == component[v] && map[i] == old_value)
                map[i] = x;
    }
```

In flood fill applications, the graph $G=(V,E)$ is usually a grid with vertices
representing 2D coordinates, $V \subset \mathbb{N}^2$. Two vertices
$v_{i,j},v_{i',j'} \in V$ are connected if $|i-i'| + |j-j'| = 1$.

The code below implements a special DFS traversal for this particular input. The
mapping $m:V\rightarrow \mathbb{N}$ is coded as a matrix, and is enough to
represent the graph.

```c
    #define MAX_DIMENSION  1000

    typedef struct {
        int i, j;
    } coord_t;

    coord_t step[] = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
    const int num_steps = 4;

    void flood_fill(int matrix[][], int n, int m, coord_t v, int x) {
        coord_t c;
        stack_t stack; /* assume a stack of coord_t */
        int s, old_value;

        old_value = matrix[v.i][v.j];
        init_stack(&stack);
        push(&stack, v);
        matrix[v.i][v.j] = x;
        while (!empty_stack(&stack)) {
            v = pop(&stack);
            for (s = 0; s < num_steps; s++) {
                c.i = v.i + step[s].i;
                c.j = v.j + step[s].j;
                if (c.i >= 0 && c.i < n && c.j >= 0 && c.j < m &&
                        matrix[c.i][c.j] == old_value) {
                    psuh(&stack, c);
                    matrix[c.i][c.j] = x;
                }
            }
        }
    }
```