# Union-Find Disjoint Set

Union-find disjoint sets are a type of grouping where each element is part of a
single group -- therefore, the sets are disjoint. They are very efficient when
performing set union.

Let $V=(v_0, \ldots, v_{n-1})$ be a vector with $n$ elements. We use a
fixed-size vector to indicate the set of each element $v_i \in V$.  Instead of
using a unique identifier for set $S \subseteq V$, we identify $S$ by the index
of any of its contained elements $v \in S$. Exactly one element of $S$ will use
its own index to indicate $S$, so let it be called the head of set $S$.  When we
retrieve the set of an element, we update its reference so that it points to the
head of the set.  To join two sets, we make the head of one set point to the
head of the other.  These operations are quite efficient, and run in
$o(\log{n})$.

```c
const int MAX_SIZE = 10000;
int set[MAX_SIZE];

void init(int set[], int size) {
    int i;
    for (i = 0; i < size; i++) {
        set[i] = i;
    }
}

int find(int set[], int elem) {
    if (set[elem] == elem) return elem;
    set[elem] = find(set[elem]);
    return set[elem];
}

int union(int set[], int a, int b) {
    a = find(set, a);
    set[a] = find(set, b);
    return set[a];
}
```
