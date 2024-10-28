# Graph Representation

A graph $G = (V, E)$ is a structure that contains a set of vertices $V$ and a
set of edges $E \subseteq V\times V$, each edge connecting two vertices.  In a
weighted graph, we define a function $w : E \rightarrow \mathbb{R}$ that denotes
the weight $w(e)$ of an edge $e \in E$. In an undirected graph, the order of the
vertices is irrelevant: $(u,v)\in E$ implies that $(v,u)\in E$, $\forall u, v
\in V$. Otherwise, we have a directed graph.

## Matrix

In a matrix representation, we store the graph as a $|V| \times |V|$ matrix $M$
where $m_{i, j} = 1$ if $(i, j) \in E$ but 0 otherwise, for each $i, j \in V$.
Accessing a specific edge $(i, j) \in E$ is done in $O(1)$, but traversing the
entire graph takes $O(V^2)$. It is ideal for dense graphs where $|E| \approx
|V|^2$.

```c
#define MAX_VERTICES  1000

typedef struct {
    char edge[MAX_VERTICES][MAX_VERTICES];
    double weight[MAX_VERTICES][MAX_VERTICES];
    int num_vertices;
} graph_t;

void init(graph_t *graph, int num_vertices) {
    int i, j;
    graph->num_vertices = num_vertices;
    for (i = 0; i < num_vertices; i++) {
        for (j = 0; j < num_vertices; j++) {
            graph->edge[i][j] = 0;
        }
    }
}

double * get_weight(graph_t *graph, int i, int j) {
    return graph->edge[i][j] ? &graph->weight[i][j] : NULL;
}

void traverse(graph_t *graph) {
    int i, j;
    for (i = 0; i < graph->num_vertices; i++) {
        for (j = 0; j < graph->num_vertices; j++) {
            if (graph->edge[i][j]) {
                printf("w(%d, %d) = %f\n", i, j, graph->weight[i][j]);
            }
        }
    }
}
```

## Adjacency List

In an adjacency list representation, we store a list $E(v)$ of edges for each
vertex $v \in V$ in the graph. We use fixed vectors to represent each list, but
other structures could be used as well. Accessing a specific edge $(i, j) \in E$
for vertices $i, j \in V$ is done in $O(|V|)$ in the worst case, but traversing
the graph takes just $O(|E|)$, which is faster than matrix representation. It is
specially advantageous in sparse graphs, where $|E| \approx |V|$.

```c
#define MAX_VERTICES  1000

struct vertex_s;

typedef struct vertex_s {
    int edge[MAX_VERTICES][MAX_VERTICES];
    double weight[MAX_VERTICES][MAX_VERTICES];
    int num_edges[MAX_VERTICES];
    int num_vertices;
} graph_t;

void init(graph_t *graph, int num_vertices) {
    int i;
    graph->num_vertices = num_vertices;
    for (i = 0; i < num_vertices; i++) {
        graph->num_edges[0] = 0;
    }
}

double * get_weight(graph_t *graph, int i, int j) {
    int e;
    for (e = 0; e < graph->num_edges[i]; e++) {
        if (graph->edge[i][e] == j) return &graph->weight[i][e];
    }
    return NULL;
}

void traverse(graph_t *graph) {
    int i, e;
    for (i = 0; i < graph->num_vertices; i++) {
        for (e = 0; e < graph->num_edges[i]; e++) {
            printf("w(%d, %d) = %f\n", i, graph->edge[i][e],
                    graph->weight[i][e]);
        }
    }
}
```
