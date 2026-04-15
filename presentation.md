# Binary Search Trees

**Computer Science Fundamentals Series**

BST invariant · Search & insertion · Balancing · Treaps · Splay trees · Order statistics

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [The BST Invariant](#slide-02--the-bst-invariant)
2. [Search](#slide-03--search)
3. [Insertion](#slide-04--insertion)
4. [Deletion](#slide-05--deletion)
5. [In-Order Successor & Predecessor](#slide-06--in-order-successor--predecessor)
6. [BST Traversals](#slide-07--bst-traversals)
7. [BST from Sorted Array](#slide-08--bst-from-sorted-array)
8. [Complexity Analysis](#slide-09--complexity-analysis)
9. [The Degeneration Problem](#slide-10--the-degeneration-problem)
10. [Why Balancing Matters](#slide-11--why-balancing-matters)
11. [Randomised BSTs](#slide-12--randomised-bsts)
12. [Treaps](#slide-13--treaps)
13. [Augmented BSTs -- Order Statistics Tree](#slide-14--augmented-bsts--order-statistics-tree)
14. [Augmented BSTs -- Interval Tree](#slide-15--augmented-bsts--interval-tree)
15. [Self-Balancing Preview -- AVL Trees](#slide-16--self-balancing-preview--avl-trees)
16. [Self-Balancing Preview -- Red-Black Trees](#slide-17--self-balancing-preview--red-black-trees)
17. [Splay Trees](#slide-18--splay-trees)
18. [BST Iterator Design](#slide-19--bst-iterator-design)
19. [Applications](#slide-20--applications)
20. [Summary & Further Reading](#slide-21--summary--further-reading)

---

## Slide 02 -- The BST Invariant

### Definition

A Binary Search Tree is a rooted binary tree where every node satisfies the **BST invariant**:

- All keys in the **left** subtree are **strictly less** than the node's key
- All keys in the **right** subtree are **strictly greater** than the node's key
- Both left and right subtrees are themselves valid BSTs

### Node structure

```
class Node:
    key:    int
    value:  any
    left:   Node | null
    right:  Node | null
```

### Visual example

```
        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4   7 13
```

Every path from root to leaf maintains sorted order when projected onto the key axis.

> The invariant gives us `O(log n)` expected search time -- each comparison eliminates roughly half the remaining nodes, just like binary search on a sorted array.

---

## Slide 03 -- Search

### Algorithm

Starting at the root, compare the target key `k` with the current node's key:

- `k == node.key` -- found, return node
- `k < node.key` -- recurse into left subtree
- `k > node.key` -- recurse into right subtree
- `node == null` -- key not present

### Pseudocode

```python
def search(node, key):
    if node is None:
        return None
    if key == node.key:
        return node
    elif key < node.key:
        return search(node.left, key)
    else:
        return search(node.right, key)
```

### Iterative variant

```python
def search_iterative(node, key):
    while node is not None:
        if key == node.key:
            return node
        elif key < node.key:
            node = node.left
        else:
            node = node.right
    return None
```

> Iterative search avoids call-stack overhead -- preferred in production for deep trees.

---

## Slide 04 -- Insertion

### Algorithm

Insertion follows the same path as search. When we reach a null pointer, we create a new node there.

```python
def insert(node, key, value):
    if node is None:
        return Node(key, value)
    if key < node.key:
        node.left = insert(node.left, key, value)
    elif key > node.key:
        node.right = insert(node.right, key, value)
    else:
        node.value = value  # duplicate key: update
    return node
```

### Insertion order matters

Inserting `[4, 2, 6, 1, 3, 5, 7]` produces a balanced tree of height 2.

Inserting `[1, 2, 3, 4, 5, 6, 7]` produces a degenerate chain of height 6.

> The shape of a BST is entirely determined by insertion order. This is the fundamental motivation for self-balancing trees.

---

## Slide 05 -- Deletion

### Three cases

| Case | Description | Action |
|------|------------|--------|
| **Leaf node** | No children | Remove the node directly |
| **One child** | One subtree | Replace node with its child |
| **Two children** | Both subtrees | Replace node with its in-order successor (or predecessor), then delete the successor |

### Pseudocode

```python
def delete(node, key):
    if node is None:
        return None
    if key < node.key:
        node.left = delete(node.left, key)
    elif key > node.key:
        node.right = delete(node.right, key)
    else:
        if node.left is None:
            return node.right
        if node.right is None:
            return node.left
        # Two children: replace with in-order successor
        successor = find_min(node.right)
        node.key = successor.key
        node.value = successor.value
        node.right = delete(node.right, successor.key)
    return node
```

> Deletion with two children is `O(h)` where `h` is the tree height -- finding the successor requires walking down the right subtree.

---

## Slide 06 -- In-Order Successor & Predecessor

### In-order successor

The node with the **smallest key greater** than a given node's key.

**Case 1 -- right subtree exists:** successor is the leftmost node in the right subtree.

**Case 2 -- no right subtree:** walk up via parent pointers until you find an ancestor where the node is in the left subtree.

```python
def find_successor(node):
    if node.right:
        return find_min(node.right)
    parent = node.parent
    while parent and node == parent.right:
        node = parent
        parent = parent.parent
    return parent
```

### In-order predecessor

The mirror: largest key smaller than the given node.

- **Right subtree exists:** rightmost node in the left subtree
- **No left subtree:** walk up until the node is in the right subtree of an ancestor

> Successor and predecessor are essential for deletion (two-child case), range queries, and iterator design.

---

## Slide 07 -- BST Traversals

### Four traversal orders

| Order | Visit sequence | Output for example tree |
|-------|---------------|------------------------|
| **In-order** | left, node, right | 1, 3, 4, 6, 7, 8, 10, 13, 14 |
| **Pre-order** | node, left, right | 8, 3, 1, 6, 4, 7, 10, 14, 13 |
| **Post-order** | left, right, node | 1, 4, 7, 6, 3, 13, 14, 10, 8 |
| **Level-order** | BFS by depth | 8, 3, 10, 1, 6, 14, 4, 7, 13 |

### In-order traversal yields sorted output

```python
def inorder(node, result):
    if node is None:
        return
    inorder(node.left, result)
    result.append(node.key)
    inorder(node.right, result)
```

> **Key insight:** in-order traversal of a BST always produces keys in ascending sorted order. This is the foundation of tree-sort (`O(n log n)` average, `O(n^2)` worst).

---

## Slide 08 -- BST from Sorted Array

### Problem

Given a sorted array, build a **height-balanced** BST (minimise height).

### Algorithm

Pick the middle element as root. Recursively build the left subtree from the left half and the right subtree from the right half.

```python
def sorted_array_to_bst(arr, lo, hi):
    if lo > hi:
        return None
    mid = (lo + hi) // 2
    node = Node(arr[mid])
    node.left  = sorted_array_to_bst(arr, lo, mid - 1)
    node.right = sorted_array_to_bst(arr, mid + 1, hi)
    return node
```

### Complexity

- **Time:** `O(n)` -- each element visited exactly once
- **Space:** `O(log n)` -- recursion stack depth equals tree height

### Result properties

- Height = `floor(log2(n))`
- All leaves at depth `h` or `h - 1`
- Equivalent to binary search's decision tree

> This is the optimal BST construction for uniformly distributed queries. For skewed query distributions, see **optimal BSTs** (Knuth's algorithm, dynamic programming).

---

## Slide 09 -- Complexity Analysis

### Time complexity by operation

| Operation | Best case | Average case | Worst case |
|-----------|----------|-------------|-----------|
| **Search** | `O(1)` | `O(log n)` | `O(n)` |
| **Insert** | `O(1)` | `O(log n)` | `O(n)` |
| **Delete** | `O(1)` | `O(log n)` | `O(n)` |
| **Min / Max** | `O(1)` | `O(log n)` | `O(n)` |
| **Successor** | `O(1)` | `O(log n)` | `O(n)` |
| **In-order traversal** | `O(n)` | `O(n)` | `O(n)` |

### Space complexity

- **Storage:** `O(n)` for `n` nodes
- **Recursive operations:** `O(h)` stack space where `h` is the height
- **Balanced tree:** `h = O(log n)`
- **Degenerate tree:** `h = O(n)`

### Average-case analysis

A BST built from `n` random insertions has expected height `O(log n)`. Specifically, the expected depth of a node is approximately `1.39 * log2(n)` -- the same constant as quicksort's expected comparisons.

> The `O(log n)` average case assumes **random insertion order**. In practice, data often arrives sorted or nearly sorted, producing the worst case. This is why we need balancing.

---

## Slide 10 -- The Degeneration Problem

### What goes wrong

When keys are inserted in sorted (or reverse-sorted) order, each new node becomes a right (or left) child of the previous one. The tree degenerates into a **linked list**.

```
Insert 1, 2, 3, 4, 5:

1                     Balanced alternative:
 \                          3
  2                        / \
   \                      1   4
    3                      \   \
     \                      2   5
      4
       \
        5

Height = n - 1          Height = 2
```

### Performance impact

| Tree shape | Height | Search time |
|-----------|--------|------------|
| Perfectly balanced | `log2(n)` | `O(log n)` |
| Random insertions | `~1.39 * log2(n)` | `O(log n)` |
| Sorted insertions | `n - 1` | `O(n)` |

### Real-world triggers

- Auto-incrementing IDs inserted in order
- Alphabetically sorted data (names, cities)
- Time-series data with monotonic timestamps
- Sequential log entries

> A degenerate BST offers **no advantage over a linked list** but uses more memory (two child pointers per node). This is the central motivation for self-balancing BSTs.

---

## Slide 11 -- Why Balancing Matters

### The guarantee

A **balanced** BST maintains `h = O(log n)` after every insertion and deletion, ensuring **worst-case** `O(log n)` operations.

### Balance criteria comparison

| Structure | Balance criterion | Height bound |
|-----------|-----------------|-------------|
| **AVL tree** | Heights of children differ by at most 1 | `1.44 * log2(n)` |
| **Red-Black tree** | No path is more than 2x the shortest | `2 * log2(n)` |
| **B-tree** | All leaves at same depth | `log_B(n)` |
| **Treap** | Random priorities, expected balance | `O(log n)` expected |
| **Splay tree** | No explicit criterion; amortised | `O(log n)` amortised |

### The cost of balance

Balancing requires extra work on each insert/delete -- typically **rotations**:

- **Single rotation** -- one pointer swap, `O(1)`
- **Double rotation** -- two pointer swaps, `O(1)`
- **Splaying** -- up to `O(log n)` rotations amortised

> The overhead is small: rotations are constant-time pointer operations. The payoff is enormous: guaranteed logarithmic performance regardless of input order.

---

## Slide 12 -- Randomised BSTs

### Idea

If insertion order determines tree shape, and random order gives `O(log n)` expected height, can we **simulate** random order?

### Approach 1 -- Shuffle then insert

Randomly permute the keys before insertion. Simple but requires all keys up front (offline).

### Approach 2 -- Randomised insertion

On inserting a key into a subtree of size `n`, with probability `1/(n+1)` insert it at the **root** of the subtree (using rotations). Otherwise, recurse normally.

```python
def rand_insert(node, key):
    if node is None:
        return Node(key)
    if random() < 1.0 / (size(node) + 1):
        return insert_at_root(node, key)
    if key < node.key:
        node.left = rand_insert(node.left, key)
    else:
        node.right = rand_insert(node.right, key)
    return node
```

### Properties

- Each key is equally likely to be the root of any subtree -- equivalent to a random permutation BST
- Expected height: `O(log n)`
- No worst-case guarantee -- but the probability of a bad tree is exponentially small

> Randomised BSTs are elegant in theory. In practice, **treaps** achieve the same probabilistic guarantees with a cleaner implementation.

---

## Slide 13 -- Treaps

### Definition

A treap is a BST where each node has a **key** (BST order) and a **random priority** (heap order). The tree simultaneously satisfies:

- **BST property** on keys (left < node < right)
- **Max-heap property** on priorities (parent priority >= child priority)

### Structure

```
        (G, 99)
       /       \
    (B, 72)   (K, 84)
    /    \       \
 (A, 31) (E, 55) (M, 41)
```

Keys: A < B < E < G < K < M (BST order)
Priorities: 99 > 84 > 72 > 55 > 41 > 31 (heap order from top)

### Operations

| Operation | Method | Expected time |
|-----------|--------|--------------|
| **Insert** | BST insert, then rotate up until heap property restored | `O(log n)` |
| **Delete** | Set priority to -infinity, rotate down to leaf, remove | `O(log n)` |
| **Split** | Insert key with priority infinity, left/right subtrees are the split | `O(log n)` |
| **Merge** | Compare roots' priorities, recursively merge subtrees | `O(log n)` |

> Treaps are simpler to implement than AVL or Red-Black trees. The random priorities ensure `O(log n)` expected height with high probability. Used in competitive programming and functional data structures.

---

## Slide 14 -- Augmented BSTs -- Order Statistics Tree

### Concept

An **augmented BST** stores additional information in each node to support extra queries efficiently. The augmented data must be maintainable during rotations in `O(1)`.

### Order statistics tree

Each node stores `size` -- the number of nodes in its subtree (including itself).

```
          (15, size=7)
         /             \
    (10, size=3)   (20, size=3)
    /        \      /        \
 (5, s=1) (12,s=1) (18,s=1) (25,s=1)
```

### Supported operations

| Operation | Description | Time |
|-----------|------------|------|
| `select(k)` | Find the k-th smallest element | `O(log n)` |
| `rank(key)` | Find the rank (position) of a key | `O(log n)` |

### Select algorithm

```python
def select(node, k):
    left_size = size(node.left)
    if k == left_size + 1:
        return node
    elif k <= left_size:
        return select(node.left, k)
    else:
        return select(node.right, k - left_size - 1)
```

> Order statistics trees are used in database engines for `OFFSET`/`LIMIT` queries and in computational geometry for rank-based operations.

---

## Slide 15 -- Augmented BSTs -- Interval Tree

### Problem

Given a set of intervals `[lo, hi]`, efficiently find all intervals that overlap a query interval or point.

### Augmentation

Each node stores an interval `[lo, hi]` as its key (ordered by `lo`), plus `max_hi` -- the maximum `hi` value in its subtree.

```
         [15, 20] max=30
        /                \
  [10, 30] max=30    [17, 19] max=20
  /                       \
[5, 8] max=8         [20, 20] max=20
```

### Overlap query

```python
def query_overlap(node, lo, hi):
    if node is None:
        return []
    results = []
    if overlaps(node.interval, (lo, hi)):
        results.append(node.interval)
    if node.left and node.left.max_hi >= lo:
        results += query_overlap(node.left, lo, hi)
    if node.right and node.right.interval.lo <= hi:
        results += query_overlap(node.right, lo, hi)
    return results
```

### Applications

- Scheduling and calendar conflict detection
- Genomics -- overlapping gene regions
- Computational geometry -- segment intersection
- Network routing -- IP range lookups

> Interval trees answer overlap queries in `O(log n + k)` where `k` is the number of overlapping intervals. Without augmentation, you would need `O(n)` per query.

---

## Slide 16 -- Self-Balancing Preview -- AVL Trees

### Definition

An AVL tree (Adelson-Velsky and Landis, 1962) is a BST where for every node, the heights of the left and right subtrees differ by at most 1.

### Balance factor

```
balance_factor(node) = height(left) - height(right)
```

Valid values: `-1, 0, +1`. Any other value triggers rebalancing.

### Rotations

| Imbalance | Rotation | Description |
|-----------|----------|------------|
| Left-Left | Right rotation | Single rotation at the unbalanced node |
| Right-Right | Left rotation | Single rotation at the unbalanced node |
| Left-Right | Left-Right rotation | Rotate left child left, then node right |
| Right-Left | Right-Left rotation | Rotate right child right, then node left |

### Properties

- Height bound: `h < 1.44 * log2(n + 2)`
- Search: `O(log n)` worst case
- Insert: `O(log n)` -- at most 2 rotations
- Delete: `O(log n)` -- up to `O(log n)` rotations

> AVL trees provide **stricter balance** than Red-Black trees, making searches slightly faster. The trade-off is more rotations on insert/delete. Preferred when reads dominate writes.

---

## Slide 17 -- Self-Balancing Preview -- Red-Black Trees

### Definition

A Red-Black tree is a BST where each node is coloured red or black, satisfying five properties:

1. Every node is red or black
2. The root is black
3. Every null leaf is black
4. Red nodes have only black children (no two reds in a row)
5. All paths from a node to its null descendants have the same **black-height**

### Height guarantee

The longest path (alternating red-black) is at most twice the shortest (all black):

`h <= 2 * log2(n + 1)`

### Practical significance

| Feature | AVL | Red-Black |
|---------|-----|-----------|
| Height bound | `1.44 * log2(n)` | `2 * log2(n)` |
| Search speed | Slightly faster | Slightly slower |
| Insert rotations | Up to 2 | Up to 2 |
| Delete rotations | Up to `O(log n)` | Up to 3 |
| Use cases | Read-heavy | Insert/delete-heavy |

> Red-Black trees are the default balanced BST in most standard libraries: `std::map` (C++), `TreeMap` (Java), `SortedDictionary` (C#). Their simpler deletion rebalancing makes them practical for general-purpose use.

---

## Slide 18 -- Splay Trees

### Idea

A splay tree is a self-adjusting BST that moves the most recently accessed node to the root via a sequence of rotations called **splaying**. No balance metadata stored.

### Splay operations

| Pattern | Name | Action |
|---------|------|--------|
| Node is child of root | **Zig** | Single rotation |
| Node and parent are both left (or both right) children | **Zig-zig** | Rotate parent first, then node |
| Node is left child of right child (or vice versa) | **Zig-zag** | Rotate node twice (like AVL double rotation) |

### Amortised analysis

- No single operation is guaranteed `O(log n)`
- But any sequence of `m` operations on an `n`-node splay tree takes `O(m * log n)` total
- **Amortised cost per operation:** `O(log n)`

### Advantages

- No extra storage (no height, colour, size, or priority fields)
- Adapts to access patterns -- frequently accessed elements stay near the root
- **Working-set property:** if you access `k` distinct elements in a window, those operations cost `O(log k)` amortised, not `O(log n)`

> Splay trees excel in caches and access-pattern-skewed workloads. They are suboptimal for uniform random access due to constant restructuring overhead.

---

## Slide 19 -- BST Iterator Design

### Problem

Provide an iterator that returns BST elements in sorted (in-order) order, using `O(h)` space and `O(1)` amortised time per `next()` call.

### Stack-based approach

```python
class BSTIterator:
    def __init__(self, root):
        self.stack = []
        self._push_left(root)

    def _push_left(self, node):
        while node:
            self.stack.append(node)
            node = node.left

    def has_next(self):
        return len(self.stack) > 0

    def next(self):
        node = self.stack.pop()
        self._push_left(node.right)
        return node.key
```

### Complexity

- **Space:** `O(h)` -- stack holds at most one node per level
- **Time:** `O(1)` amortised per `next()` -- each node is pushed and popped exactly once across the full traversal
- **Worst-case single call:** `O(h)` -- when `next()` must descend a full left spine

### Morris traversal alternative

Uses `O(1)` extra space by temporarily modifying tree pointers (threading). Restores the tree after traversal. Not suitable for concurrent access.

> The stack-based iterator is a classic interview question and the standard implementation in most BST libraries. It decouples traversal from processing.

---

## Slide 20 -- Applications

### Symbol tables and dictionaries

BSTs implement the ordered symbol table ADT: `put(key, val)`, `get(key)`, `delete(key)`, `min()`, `max()`, `floor()`, `ceiling()`, `rank()`, `select()`.

Used in: compilers (symbol tables), interpreters, configuration stores.

### Database indexing

B-trees (a generalisation of BSTs with high branching factor) are the default index structure in all major relational databases. BST concepts (invariant, rotations, balance) directly underpin B-tree operations.

### Auto-complete and prefix search

BSTs on strings (tries are a related structure) support prefix-based lookup. A balanced BST of words answers "what words come between X and Y?" in `O(log n + k)`.

### Computational geometry

- **k-d trees** -- BSTs partitioning k-dimensional space
- **Interval trees** -- scheduling, genomics
- **Range trees** -- orthogonal range queries

### Operating systems

- Process scheduling (Linux CFS uses a Red-Black tree keyed by virtual runtime)
- Memory management (VM area tracking via Red-Black tree in the Linux kernel)
- File systems (ext4 extent trees, Btrfs B-trees)

> BSTs are not just an academic exercise. They are the backbone of ordered data access in virtually every system you use.

---

## Slide 21 -- Summary & Further Reading

### Key takeaways

- The BST invariant (left < node < right) enables `O(log n)` search, insert, and delete -- but only when the tree is balanced
- Degenerate BSTs reduce to linked lists with `O(n)` operations -- insertion order is everything
- Balancing strategies (AVL, Red-Black, treaps, splay) restore `O(log n)` guarantees at minimal overhead
- Augmented BSTs (order statistics, interval trees) extend the BST to answer richer queries without changing asymptotic complexity
- Treaps combine BST + heap properties using random priorities for expected `O(log n)` height
- Splay trees achieve amortised `O(log n)` through access-driven restructuring -- optimal for skewed workloads
- The stack-based BST iterator provides `O(h)` space, `O(1)` amortised in-order traversal
- BSTs underpin databases (B-trees), operating systems (Red-Black trees), and compilers (symbol tables)

### Recommended reading

| Source | Description |
|--------|------------|
| **Sedgewick & Wayne** | *Algorithms*, 4th ed. -- definitive BST treatment with Java implementations |
| **CLRS** | *Introduction to Algorithms* -- formal proofs for Red-Black trees, augmented BSTs, order statistics |
| **Skiena** | *The Algorithm Design Manual* -- practical perspective on when to use which tree |
| **Sleator & Tarjan** | "Self-Adjusting Binary Search Trees" -- the original splay tree paper (1985) |
| **MIT 6.006** | Erik Demaine's lectures on BSTs -- free on YouTube, excellent visualisations |
