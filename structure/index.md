---
is_index: true
order: 1
---

# Data structures

This section contains basic algorithms for general data structures and their
time complexity. They are used to insert and query data based on an integer key
field, and vary on their efficiency. Modern programming languages already
provide most of these structures as builtin types or classes.

One structure in particular that does not appear here is binary tree. Instead of
presenting a simple version of binary tree, which is unbalanced and possibly not
efficient, I decided to present a B-tree, which is rarely used in contests, but
is a safer alternative. (Other approaches could be AVL trees and Red-Black
trees, but I have a hard time remembering all the rotation cases.)


* [Vectors](./vector.md): binary search, merge and quick sort, order statistic
* [Linked lists](./linked-list.md): search, insertion and removal for simple
  unbounded lists
* [Stacks](./stack.md): fixed-size stack operations
* [Queues](./queue.md): fixed-size queue operations, queues of minima
* [Hash tables](./hashtable.md): fixed-size tables with simple collision
  treatment
* [Heaps](./heap.md): search, insertion and removal for priority queues
* [B-trees](./b-tree.md): search and insertion for balanced $k$-ary trees
* [Segment trees](./segment-tree.md): query and update of data intervals
* [Union-find disjoint sets](./set.md): search and merge of sets
