---
order: 6
---

# Network Flow

Let $G=(V,E)$ be a directed graph representing a network flow, with vertices $s,
t \in V$ denoting the source and the sink of $G$ respectively.  The function $c:
E \rightarrow \mathbb{N}$ denotes the flow capacity along the edge. We define a
_flow_ along the edges as a function $f: E \rightarrow \mathbb{N}$ that respects
the following constraints:

**Capacity constraint** $f(e) \leq c(e), \forall e \in E$, that is, the flow
along an edge cannot be higher than its capacity.

**Flow constraint** $\sum_{(x, v) \in E} f(x, v) = \sum_{(v, y) \in E} f(v, y),
\forall v \in V \setminus \{s,t\}$, that is, all flow that enters a vertex $v$
must exit it (except at the source and at the sink).

We define the total flow $F(G)$ of a network $G$ as the sum of all flow leaving
its source $s$ and entering its sink $t$:

$$F(G) = \sum_{(s, x) \in E} f(s, x) = \sum_{(y, t) \in E} f(y, t)$$

## Maximum network flow

Given a directed graph representing a network flow with a given capacity
function and source and sink vertices, compute the maximum flow that can reach
the sink.

**Input** A directed graph $G=(V,E)$, a capacity function $c : E \rightarrow
\mathbb{N}$, and source and sink vertices \
**Output** A flow $f: E \rightarrow \mathbb{N}$ whose total $F(G)$ is maximal \
**Time** $O(|V| \cdot |E|^2)$

This algorithm is attributed to Edmond & Karp. The idea is to find a route from
source to sink that increases the flow, and to continue to add new flow routes
until we cannot increase it further. To find a route, we simply use
[breadth-first search](./traversal.md#breadth-first-search) on the network,
observing that two vertices $v, u \in V$ are connected if and only if the
current flow in $(v, u)$ is lower than the edge's capacity $c(v, e)$. After the
search, we have an _augmenting path_, a flow route from source to sink that is
guaranteed to increase the total flow.

With an augmenting path, we determine the maximum value we can route through
this path (its bottleneck) and increase the flow $f$ along its edges by this
amount. We also decrease the flow in the reverse direction by the same amount,
even though there is no reverse connection between the vertices. This allows the
algorithm to revert part of flow allocated in case a new augmenting path can
make better use of it in the future.

Each augmenting path saturates the capacity of an edge. Once an edge is
saturated, it may have its flow reverted, but once it is saturated again, the
new augmenting path will be longer than the previous one (that is, it will have
at least one more vertex in the path). Summing up, the algorithm can find
in the worst case up to $|E|·|V|$ augmenting paths. Since each breath-first
search runs in $O(|E|)$, Edmond-Karp's algorithm runs in $O(|V|\cdot |E|^2)$.

The algorithm below use
[adjacency lists](./representation.md#adjacency-list) to represent a graph,
and a [matrix](./representation.md#matrix) to represent the flow between two
vertices.

```c
#define MAX_VERTICES  1000
#define MAX_EDGES     (MAX_VERTICES)  /* edges per vertex */
#define INFINITY      (MAXINT >> 1)
#define MIN(x, y)     ((x) < (y) ? (x) : (y))

typedef struct {
    int edge[MAX_VERTICES][MAX_EDGES];
    int capacity[MAX_VERTICES][MAX_EDGES];
    int num_edges[MAX_VERTICES];
    int num_vertices;
} graph_t;

int flow[MAX_VERTICES][MAX_VERTICES];  // matrix representation

int maximum_flow(graph_t *graph, int source, int sink, int flow[][]) {
    int v, u, e, minimum, total_flow, found_sink;
    int maximum[MAX_VERTICES][MAX_VERTICES];
    int flow[MAX_VERTICES][MAX_VERTICES];

    for (v = 0; v < graph->num_vertices; v++) {
        for (u = 0; u < graph->num_vertices; u++) {
            maximum[v][u] = flow[u][v] = 0;
        }
        for (e = 0; e < graph->num_edges[v]; e++) {
            maximum[v][graph->edge[v][e]] = graph->capacity[v][e];
        }
    }
    total_flow = 0;

    while (1) {
        /* Find path linking source to sink */
        int prior[MAX_VERTICES], queue[MAX_VERTICES], head, tail, found;
        for (v = 0; v < graph->num_vertices; v++) {
            prior[v] = -1;
        }
        queue[0] = source, head = 0, tail = 1, found = 0;
        while (!found && head < tail) {
            v = queue[head++];
            for (e = 0; !found && e < graph->num_edges[v]; e++) {
                int u = graph->edge[v][e];
                if (prior[u] == -1 && u != source
                        && flow[v][u] < maximum[v][u]) {
                    queue[tail++] = u;
                    prior[u] = v;
                    found = (u == sink);
                }
            }
        }
        if (!found) return total_flow;

        /* Update flow network with path's capacity */
        minimum = INFINITY;
        for (u = sink; prior[u] != -1; u = prior[u]) {
            v = prior[u];
            minimum = MIN(minimum, maximum[v][u] - flow[v][u]);
        }
        total_flow += minimum;
        for (u = sink; prior[u] != -1; i = prior[u]) {
            flow[prior[u]][u] += minimum;
            flow[u][prior[u]] -= minimum;
        }
    }
}
```
