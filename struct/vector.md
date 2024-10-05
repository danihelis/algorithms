# Vectors

Fixed-size vectors are used as storage for finite sequences of values, and may
be used to implement more complex data structures. In many applications, we are
interested in an ordered sequence of data. The algorithms in this section deal
with that.

We will represent a vector $V$ of size $n$ using the notation $V=(v_0,
\ldots, v_{n-1})$, where $v_i$ denotes an element from the vector. A vector is
sorted if $v_i \leq v_j, \forall i \leq j$.

```c
const int MAX_SIZE = 10000;
int vector[MAX_SIZE];
```


## Binary Search

Given a sorted vector and an element, find the index of the element inside the
vector. If the element is not present, find an index where the element might
be inserted while maintaining the vector sorted.

**Input** A sorted vector $V$ of length $n$, and an element $x$ \
**Output** The index $i$ such that $v_i = x$, if $x \in V$; otherwise,
an index $i$ such that $v_j \leq x, \forall j < i$, and $v_k \geq x,
\forall k > i$ \
**Time** $O(\log{n})$

The algorithm works recursively for a range $[a, b)$ of the vector $V$, with $0
\leq a < b \leq n$. Let $m = (a + b) / 2$ denote the middle of the range.  If
$v_m = x$, the desired index is $m$. If not, we check the range $[a, m)$ if $x <
v_m$, or $[m + 1, b)$ otherwise. If $a = b$, then $a$ is the index where $x$ is
supposed to be inserted.

```c
int binary_search(int vector[], int size, int value) {
    int start = 0, end = size;
    while (start < end) {
        int middle = (start + end) / 2;
        int compare = vector[middle] - value;
        if (compare == 0) return middle;
        else if (compare > 0) end = middle;
        else start = middle + 1;
    }
    return start;
}
```

To insert a new element in the sorted vector, we use the index returned by
`binary_search` and shift all larger elements to give space to the new element.
The cost of insertion is $O(n)$. Notice that the cost of inserting $n$ elements
this way is $O(n^2)$.

```c
void insert(int vector[], int *size, int value) {
    int pos = binary_search(vector, *size, value);
    if (pos >= size || vector[pos] != value) {
        int i;
        for (i = *size; i > pos; i--) {
            vector[i] = vector[i - 1];
        }
        vector[pos] = value;
        (*size)++;
    }
}
```


## Merge Sort

Given an unsorted vector, sort all its elements.

**Input** An unsorted vector $V$ of length $n$ \
**Effect** The vector $V$ becomes sorted \
**Time** $O(n \log{n})$ \
**Space** $O(n)$

The algorithm works recursively for a range $[a, b)$ of the vector $V$, with
$0 \leq a < b \leq n$. Let $m = (a + b) / 2$ denote the middle of the range.
First we sort the two non-empty halves of the range, $[a, m)$ and $[m, b)$.
Then we merge these two halves, interweaving their elements as to preserve the
ordering.

Note that the algorithm uses an auxiliary vector, defined statically to avoid
stack overflow.

```c
void merge_sort(int vector[], int start, int end) {
    static int temp_vector[MAX_SIZE];
    if (end - start > 1) {
        int i, middle, first, second;
        middle = (start + end) / 2;
        first = start;
        second = middle;
        merge_sort(vector, first, middle);
        merge_sort(vector, second, end);
        i = 0;
        while (first < middle || second < end) {
            if (second == end || (first < middle
                    && vector[first] < vector[second])) {
                temp_vector[i++] = vector[first++];
            } else temp_vector[i++] = vector[second++];
        }
        for (i--; i >= 0; i--) {
            vector[start + i] = temp_vector[i];
        }
    }
}

void sort(elem_t vector[], int size) {
    merge_sort(vector, 0, size);
}
```


## Quick Sort

Given an unsorted vector, sort all its elements.

**Input** An unsorted vector $V$ of length $n$ \
**Effect** The vector becomes sorted \
**Time** $O(n \log{n})$, on average \
**Space** $O(1)$

The algorithm works recursively for a range $[a, b)$ of the vector $V$, with
$0 \leq a < b \leq n$. First, we choose any element of the vector as pivot.
Let its index be $p$, with $a \leq p < b$. Then we swap some elements of the
vector until we have $v_i \leq v_p, \forall i < p$,
and $v_j > v_p, \forall j > p$. Finally we sort the non-empty ranges
$[a, p)$ and $[p + 1, b)$.

```c
void swap(elem_t vector[], int i, int j) {
    int element = vector[i];
    vector[i] = vector[j];
    vector[j] = element;
}

int find_pivot(int vector[], int start, int end) {
    int lower = start + 1, higher = end - 1;
    while (lower <= higher) {
        while (lower <= higher && vector[lower] <= vector[start]) lower++;
        while (lower <= higher && vector[higher] > vector[start]) higher--;
        if (lower < higher) swap(vector, lower++, higher--);
    }
    if (lower - 1 > start) swap(start, lower - 1);
    return lower - 1;
}

void quick_sort(elem_t vector[], int start, int end) {
    if (end - start > 1) {
        int pivot = find_pivot(vector, start, end);
        quick_sort(vector, start, pivot);
        quick_sort(vector, pivot + 1, end);
    }
}

void sort(elem_t vector[], int size) {
    quick_sort(vector, 0, size);
}
```

The time complexity of the algorithm above depends on how well the pivot is
chosen. In the worst case, the algorithm runs in $O(n^2)$. Thus, it is
recommended to use the sorting function available in the library of your
programming language, since this drawback is likely to be corrected there.
In ANSI C, this function is called
    [qsort](http://www.cplusplus.com/reference/cstdlib/qsort/>).

```c
#include <stdlib.h>

int compare(const void *a, const void *b) {
    return *((int *) a) - *((int *) b);
}

void sort(int vector[], int size) {
    qsort(vector, size, sizeof int, compare);
}
```


## Order Statistics

Given an unsorted vector, find its $k$-th smallest element.

**Input** An unsorted vector $V$ of length $n$, and an index $k$ such that
$0 \leq k < n$ \
**Output** The $k$-th smallest element of the vector \
**Time** $O(n)$, on average

The algorithm is similar to `quick_sort`. It works recursively for a range $[a,
b)$ of the vector $V$, with $0 \leq a < b \leq n$.  First we obtain a pivot at
position $p$ such that $v_i \leq v_p, \forall i < p$, and $v_j > v_p,
\forall j > p$.  If $p = k$, then we have found it.  If not, we search for
the $k$-th element in the range $[a, p)$ if $k < p$, or $[p + 1, b)$ otherwise.

```c
int kth_index(int vector[], int start, int end, int k) {
    int pivot = find_pivot(vector, start, end);
    if (pivot == k) return pivot;
    else if (k < pivot) return kth_index(vector, start, pivot, k);
    else return kth_index(vector, pivot + 1, end, k);
}

int kth_element(int vector[], int size, int k) {
    return vector[kth_index(vector, 0, size, k)];
}
