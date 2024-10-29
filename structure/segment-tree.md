---
order: 8
---

# Segment Tree

Segment trees are binary trees used to compute information from any range over a
sequence.  Let $S = (s_0, ..., s_{n-1})$ be a sequence of $n$ elements. The root
node of $S$'s segment tree represents the range $[0, n)$, and it branches out
into two sub-trees: one for range $[0, n/2)$, and another for $[n/2, n)$. The
leaves of the segment tree represent the range $[i, i+1), \forall i: 0 \leq i <
n$, that is, each element $s_i \in S$.

Segment trees are particularly effective when we need to evaluate a combination
of values from subsequence $(s_a, \ldots, s_{b-1}) \subseteq S$.


## Sequence evaluation

In order to evaluate the combination of values from a sequence $S \in \Sigma^*$,
we must first define an evaluation function $f: \Sigma \times \mathbb{N}
\rightarrow E$. Given a single element $s \in \Sigma$ and how many times it
appears in a subsequence, $f$ produces an evaluation from set $E$. We must also
define a binary operator $\oplus$ that combines two evaluations $e_1, e_2 \in
E$.  With these functions, we can evaluate a subsequence $S_{a,b} = (s_a,
\ldots, s_{b-1})$ by computing $f(s_a, 1) \oplus \ldots \oplus f(s_{b-1}, 1)$.
Note that

$$\underbrace{f(s, 1) \oplus \ldots \oplus f(s, 1)}_{n \text{ values}} = f(s, 1)
\oplus^{n-1} f(s, 1) = f(s, n)$$

The evaluation function $f$ must be defined according to the application. Here
are two examples:

* Each element of the sequence is either white $\alpha$ or black $\beta$, so
  $\Sigma = \{\alpha, \beta\}$. We want to compute the number of white elements
  in an interval. We define the evaluation function $f: \{\alpha, \beta\} \times
  \mathbb{N} \rightarrow \mathbb{N}$ as

  $$f(s, n) = \begin{cases}
        n & \text{if } s = \alpha \\
        0 & \text{otherwise} \\
  \end{cases}$$

  and $n_1 \oplus n_2 = n_1 + n_2$.

* Each element of the sequence is an integer $x \in \mathbb{Z}$.  We want to
  compute the signal resulting from the multiplication of all numbers in an
  interval. We define the evaluation function $f: \mathbb{Z} \times \mathbb{N}
  \rightarrow \{ +, -, 0\}$ as

  $$f(x, n) = \begin{cases}
        0 & \text{if } x = 0 \\
        + & \text{if } x > 0 \text{ or } n \text{ is even} \\
        - & \text{otherwise} \\
  \end{cases}$$

  and

  $$e_1 \oplus e_2 = \begin{cases}
        0 & \text{if } e_1 = 0 \text{ or } e_2 = 0 \\
        + & \text{if } e_1 = e_2 \\
        - & \text{otherwise} \\
  \end{cases}$$

In this section, we will assume that the types and functions below are defined.

```c
eval_t evaluate(symbol_t symbol, int quantity);
eval_t combine(eval_t e1, eval_t e2);
```


## Construction

Given a vector containing a sequence of symbols, build a segment tree covering
the whole range.

**Input** A vector $V=(s_0, \ldots, s_{n-1})$ of length $n$ \
**Output** A segment tree covering the interval $[0, n)$ \
**Time** $O(n)$

The segment tree is built only once. After that, the structure doesn't change,
only the information in the nodes may be modified. Each node contains the range
$[a, b)$ it represents, $0 \leq a < b \leq n$. It also contains an evaluation
value for the whole range and a unique symbol in case all elements in the range
are the same.

```c
typedef struct seg_s {
    int start, middle, end;
    struct seg_s *left, *right;
    symbol_t symbol;
    eval_t eval;
    char must_reevaluate, must_update_children;
} seg_t;

seg_t *tree_root;

seg_t * create_segment(int start, int end) {
    seg_t *seg = (seg_t *) malloc(sizeof (seg_t));
    seg->start = start;
    seg->end = end;
    seg->middle = (start + end) / 2;
    seg->must_reevaluate = seg->must_update_children = 0;
    seg->left = seg->right = NULL;
    return seg;
}

seg_t * build_segment(symbol_t vector[], int start, int end) {
    seg_t *seg = create_segment(start, end);
    if (end - start == 1) {
        seg->symbol = vector[start];
        seg->eval = evaluate(seg->symbol, 1);
    } else {
        seg->left = build_segment(vector, start, seg->middle);
        seg->right = build_segment(vector, seg->middle, end);
        seg->eval = combine(seg->left->eval, seg->right->eval);
    }
    return seg;
}

seg_t * build_segment_tree(symbol_t vector[], int size) {
    return build_segment(vector, 0, size);
}
```


Modification
------------

Given a segment tree, change the values of an interval to the given symbol.

**Input** A segment tree of length $n$, an interval $[a, b), 0 \leq a < b \leq
n$, and a symbol $s$ \
**Effect** Replace all elements in the interval with $s$ \
**Time** $O(\log{n})$

The strategy is quite simply. We start at the root of the segment tree. If the
interval matches the node's range, we set the entire node with the new symbol.
Otherwise, we recursively modify its sub-segments if their range is part of the
given interval.

We use delayed update propagation to amortize the costs of modification. When we
replace the values of an entire segment, we don't update its sub-segments, but
we pre-compute its evaluation. If we replace partially the values of a segment,
we update its sub-segments as needed, and indicate that its previous evaluation
is outdated. It will be recomputed only if requested.

```c
#include <assert.h>

#define MAX(a, b)   ((a) > (b) ? (a) : (b))
#define MIN(a, b)   ((a) < (b) ? (a) : (b))

void update_children(seg_t *);

void replace(seg_t *seg, int start, int end, symbol_t symbol) {
    assert(start < end && start >= seg->start && end <= seg->end);
    if (start == seg->start && end == seg->end) { /* match */
        seg->symbol = symbol;
        seg->must_update_children = 1;
        seg->evaluation = evaluate(symbol, end - start);
        seg->must_reevaluate = 0;
    } else { /* partial match */
        update_children(seg);
        if (start < seg->middle) {
            replace(seg->left, start, MIN(seg->middle, end), symbol);
        }
        if (end > seg->middle) {
            replace(seg->rigth, MAX(seg->middle, start), end, symbol);
        }
        seg->must_reevaluate = 1;
    }
}

void update_children(seg_t *seg) {
    if (seg->must_update_children) {
        replace(seg->left, seg->start, seg->middle, seg->symbol);
        replace(seg->right, seg->middle, seg->end, seg->symbol);
        seg->must_update_children = 0;
    }
}
```


## Evaluation

Given a segment tree, evaluate a particular interval contained in it.

**Input** A segment tree of length $n$, and an interval $[a, b), 0 \leq a <
b \leq n$ \
**Output** The evaluation of the values in the interval $[a, b)$ \
**Time** $O(\log{n})$

We use the same strategy as before, starting at the root node. If the interval
matches the node's range, we return the evaluation stored in it, or recompute
the evaluation, if it is outdated. Otherwise, we evaluate its sub-segments and
combine these values, if needed. We only visit the segments that are part of the
interval.

(Note that if a segment must be reevaluated, then its sub-segments were already
updated. This is because the flag `must_reevaluate` is raised only after
executing `update_children`.)

```c
eval_t evaluate_segment(seg_t *seg, int start, int end) {
    assert(start < end && start >= seg->start && end <= seg->end);
    if (start == seg->start && end == seg->end) {
        if (seg->must_reevaluate) {
            eval_t e1 = evaluate_segment(seg->left, seg->start, seg->middle);
            eval_t e2 = evaluate_segment(seg->right, seg->middle, seg->end);
            seg->eval = combine(e1, e2);
            seg->must_reevaluate = 0;
        }
        return seg->eval;
    } else {
        eval_t e1, e2;
        update_children(seg);
        if (start < seg->middle) {
            e1 = evaluate_segment(seg->left, start, MIN(seg->middle, end));
            if (end <= seg->middle) return e1;
        }
        e2 = evaluate_segment(seg->rigth, MAX(seg->middle, start), end);
        return start >= seg->middle ? e2 : combine(e1, e2);
    }
}
```
