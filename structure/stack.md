# Stacks

Stacks store data in a way that the first element inserted is the last one
retrieved. They are the natural structures for nested elements. In this section,
we implement stacks using fixed-size vectors. Another possibility is to use
[simple linked lists](./linked-list.md) with insertion and removal at start of
the list.  All operations run in $O(1)$.

```c
#include <assert.h>

const int MAX_SIZE = 10000;

typedef struct {
    int data[MAX_SIZE];
    int top;
} stack_t;

void init(stack_t *stack) {
    stack->top = 0;
}

int is_empty(stack_t *stack) {
    return stack->top == 0;
}

int is_full(stack_t *stack) {
    return stack->top == MAX_SIZE;
}

void push(stack_t *stack, int value) {
    assert(!is_full(stack));
    stack->data[stack->top++] = value;
}

int pop(stack_t *stack) {
    assert(!is_empty(stack));
    return stack->data[--stack->top];
}

int * peek(stack_t *stack) {
    return is_empty(stack) ? NULL : &stack->data[stack->top - 1];
}
```
