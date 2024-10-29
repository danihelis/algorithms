---
order: 4
---

# Queues

Queues store data in a way that the first element inserted is the first one
retrieved. They are the natural structures for buffering data that must be
processed in order. In this section, we implement queues using fixed-size
vectors. Another possibility is to use a [simple linked list](./linked-list.md)
with insertion at the end of the list and removal at the start. All operations
run in $O(1)$.

The vector in the code below represents a circular list, with the indices
wrapping around at the end of the vector. We consider the queue to be full when
the last element has an index equal to the first element index minus 1 (in
modulo).

```c
#include <assert.h>

const int MAX_SIZE = 10000;

typedef struct {
    int data[MAX_SIZE];
    int head, tail;
} queue_t;

void init(queue_t *queue) {
    queue->head = 0;
    queue->tail = 0;
}

int is_empty(queue_t *queue) {
    return queue->head == queue->tail;
}

int is_full(queue_t *queue) {
    return (queue->tail + 1) % MAX_SIZE == queue->head;
}

void enqueue(queue_t *queue, int value) {
    assert(!is_full(queue));
    queue->data[queue->tail] = value;
    queue->tail = (queue->tail + 1) % MAX_SIZE;
}

int dequeue(queue_t *queue) {
    int value = queue->data[queue->head];
    assert(!is_empty(queue));
    queue->head = (queue->head + 1) % MAX_SIZE;
    return value;
}
```


## Queues using two stacks

Implementing a queue with two [stacks](./stack.md) is more of an exercise than a
recommended practice, even though all operations run in $O(1)$ amortized time.
To enqueue an element, we push it into the first stack $E$. To dequeue an
element, we pop it from the second stack $D$, if it is not empty. If it is
empty, we pop all elements from $E$ and push them into $D$.  In this way, we
guarantee that the queue order is preserved.

```c
#include <assert.h>

const int MAX_SIZE = 10000;

typedef struct {
    int data[MAX_SIZE];
    int top;
} stack_t;

typedef struct {
    stack_t in, out;
} queue_t;

void init(queue_t *queue) {
    queue->in.top = 0;
    queue->out.top = 0;
}

int is_empty(queue_t *queue) {
    return queue->in.top == 0 && queue->out.top == 0;
}

int is_full(queue_t *queue) {
    return queue->in.top + queue->out.top == MAX_SIZE;
}

void enqueue(queue_t *queue, int value) {
    assert(!is_full(queue));
    queue->in.data[queue->in.top++] = value;
}

int dequeue(queue_t *queue) {
    assert(!is_empty(queue));
    if (queue->out.top == 0) {
        while (queue->in.top > 0) {
            int value = queue->in.data[--queue->in.top];
            queue->out.data[queue->out.top++] = value;
        }
    }
    return queue->out[--queue->out.top];
}
```


## Queues of minima

Queues of minima are queues whose smallest element can be peeked in $O(1)$. It
is different from [priority queues](./heap.md), which always retrieve the
smallest element. Queues of minima still retrieve the first element inserted.

In order to create an efficient queue of minima, we [simulate a queue using
stacks](#queues-using-two-stacks) the same way as we did before. The difference
is that we replace each original stack $S$ by two stacks $\dot{S}$ and
$\bar{S}$.  We will store the smallest values (the minima) in $\dot{S}$ and the
remaining values in $\bar{S}$.  When we would push an element $x$ into $S$, we
compare it to the top of $\dot{S}$ and either push it into $\dot{S}$ if $x$ is
smaller, or into $\bar{S}$ otherwise. We must also annotate the queue position
for $x$, so that we can dequeue it in the correct order. When we would pop an
element from $S$, we pop it from either $\dot{S}$ or $\bar{S}$ depending on the
queue position annotated. The minimum value of the queue is guaranteed to be the
top element of either the enqueue stack of minima or the dequeue stack of
minima.

```c
#include <assert.h>

const int MAX_SIZE = 10000;

typedef struct {
    int value, position;
} elem_t;

typedef struct {
    elem_t data[MAX_SIZE];
    int top;
} stack_t;

typedef struct {
    stack_t minima, others;
} pair_stack_t;

typedef struct {
    pair_stack_t in, out;
    int count;
} queue_t;

void init_pair(pair_stack_t *pair) {
    pair->minima.top = 0;
    pair->others.top = 0;
}

void init(queue_t *queue) {
    init_pair(&queue->in);
    init_pair(&queue->out);
    queue->count = 0;
}

int is_empty(queue_t *queue) {
    return queue->in.minima.top == 0 && queue->out.minima.top == 0;
}

int is_full(queue_t *queue) {
    return (queue->in.minima.top + queue->in.others.top
            + queue->out.minima.top + queue->out.others.top) == MAX_SIZE;
}

elem_t * peek_stack(stack_t *stack) {
    return stack->top == 0 ? NULL : stack->data[stack->top - 1];
}

void push(pair_stack_t *pair, elem_t element) {
    elem_t *minimum = peek_stack(&pair->minima);
    if (minimum == NULL || element.value < minimum->value) {
        pair->minima.data[pair->minima.top++] = element;
    } else {
        pair->others.data[pair->others.top++] = element;
    }
}

elem_t pop(pair_stack_t *pair, int inverted) {
    elem_t *minimum = peek_stack(&pair->minima);
    elem_t *other = peek_stack(&pair->others);
    if (other == NULL
            || (minimum->position < other->position && !inverted)
            || (minimum->position > other->position && inverted)) {
        pair->minima.top--;
        return *minimum;
    }
    pair->others.top--;
    return *other;
}

void enqueue(queue_t *queue, int value) {
    elem_t element;
    assert(!is_full(queue));
    element.value = value;
    element.position = queue->count++;
    push(&queue->in, element);
}

elem_t dequeue(queue_t *queue) {
    assert(!is_empty(queue));
    if (queue->out.minima.top == 0 && queue->out.others.top == 0) {
        while (queue->in.minima.top > 0 || queue->in.others.top > 0) {
            elem_t element = pop(&queue->in, 1 /* inverted */);
            push(&queue->out, element);
        }
    }
    return pop(&queue->out, 0 /* not inverted */);
}

int * peek_minimum(queue_t *queue) {
    elem_t *in = peek_stack(&pair->in.minima);
    elem_t *out = peek_stack(&pair->out.minima);
    if (in != NULL && (out == NULL || in->value < out->value)) {
        return &in->value;
    }
    return out != NULL ? &out->value : NULL;
}
```
