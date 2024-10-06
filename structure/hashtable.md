# Hash Tables

Hash tables are used to efficiently store and retrieve unique data.  Ideally,
these operations would run in $O(1)$, but that may not be the case. In this
section, we use a large fixed-size vector to store the data.


## Hash Function

Given an element $x$, we use a hash function to map $x$ into an index $i$, $0
\leq i < n$, where $n$ is the size of the table. If the hash function runs in
$O(1)$, we can store and retrieve $x$ at the index $i$ of the table in $O(1)$.
If two elements are mapped to the same index, however, a [collision](#collision)
will occur, requiring more computational time. To reduce the chance of
collision, we must use a large table and a good hash function that spreads
evenly the data across the indices.  How good a hash function is depends on the
particular data.

We can make a simple and universal hash function by converting the binary data
of $x$ into a large binary number, and then computing this number modulo $n$. To
lower the chances of collision, we should choose a prime number for $n$, as
suggested in the table below.

| Intended size | Hash table size |
| :---:         | :---:           |
| 1,000         | 1,009           |
| 10,000        | 10,007          |
| 100,000       | 100,003         |
| 1,000,000     | 1,000,003       |

```c
const int MAX_SIZE = 10007;

int hashcode(void *value, int value_size) {
    int index, code;
    for (index = 0, code = 0; index < value_size; index++) {
        unsigned char *byte = index + (unsigned char *) value;
        code = (code << 8 + *byte) % MAX_SIZE;
    }
    return code;
}
```


## Collision

A collision happens when two elements are mapped to the same index by a hash
function. There are many strategies to deal with it, and we present two simple
strategies. Let $n$ be the size of the hash table, and suppose that it will
store $O(n)$ elements in the worst case.

**Linked list.** We store a [simple linked list](structure/linked-list.md) in
each entry of the table, which will contain all elements that have the same hash
value. In the worst case, retrieving an element would cost $O(n)$.

**Static collision.** We use a hash function that produces a sequence of
different indices. If there is a collision with the first index, we try the next
one and so forth, until there is no collision. Given a hash function $f$, we can
create this new hash function $g$ as:
$$
    g(x, t) = f(x) + k \cdot t \mod{n},
$$
where $k$ and $n$ are relative primes. In the worst case, retrieving an element
would cost $O(n)$.


## Structure

We present two different structures for the hash table, one for each collision
strategy. In the examples below we use a simple `int` value as data, but in a
real scenario a more complex object would be stored.

**Linked list strategy**

```c
typedef struct node_s {
    int value;
    struct node_s *next;
} node_t;

node_t hashtable[MAX_SIZE];

void init(node_t table[]) {
    int i;
    for (i = 0; i < MAX_SIZE; i++) {
        table[i].next = NULL;
    }
}
```

**Static collision strategy**

```c
#include <assert.h>

typedef struct {
    int value;
    char used;
} elem_t;

elem_t hashtable[MAX_SIZE];

void init(elem_t table[]) {
    int i;
    for (i = 0; i < MAX_SIZE; i++) {
        table[i].used = 0;
    }
}

int get_index(elem_t hash[], int value) {
    int code = hashcode(&value, sizeof (int));
    int index = code;
    while (hash[index].used && hash[index].value != value) {
        index = (index + 2) % MAX_SIZE;
        assert(index != code); /* overflow */
    }
    return index;
}
```


## Insertion

Insert an element in the position computed by the hash function.

**Linked list strategy**

```c
#include <stdlib.h>

void insert(node_t table[], int value) {
    int index = hashcode(&value, sizeof (int));
    node_t *node = (node_t *) malloc(sizeof (node_t));
    node->value = value;
    node->next = table[index].next;
    table[index].next = node;
}
```

**Static collision strategy**

```c
void insert(elem_t table[], int value) {
    int index = get_index(table, value);
    table[index].value = value;
    table[index].used = 1;
}
```


## Search

Search and retrieve an element from the hash table.

**Linked list strategy**

```c
node_t * get_prior(node_t *node, int value) {
    while (node->next != NULL && node->next->value != value) {
        node = node->next;
    }
    return node;
}

int * search(node_t table[], int value) {
    int index = hashcode(&value, sizeof (int));
    node_t *prior = get_prior(&table[index], value);
    return prior->next != NULL ? &prior->next->value : NULL;
}
```

**Static collision strategy**

```c
int * search(elem_t table[], int value) {
    int index = get_index(table, value);
    return table[index].used ? &table[index].value : NULL;
}
```


## Removal

Remove an element from the hash table, if it is present.

**Linked list strategy**

```c
#include <stdlib.h>

void remove(node_t table[], int value) {
    int index = hashcode(&value, sizeof (int));
    node_t *prior = get_prior(&hash->table[index], key);
    if (prior->next != NULL) {
        node_t *node = prior->next;
        prior->next = node->next;
        free(node);
    }
}

void remove_all(node_t table[]) {
    int i;
    for (i = 0; i < MAX_SIZE; i++) {
        while (table[i].next != NULL) {
            node_t *node = table[i].next;
            table[i].next = node->next;
            free(node);
        }
    }
}
```

**Static collision strategy**

```c
void remove(elem_t table[], int value) {
    int index = get_index(table, value);
    table[index].used = 0;
}
```
