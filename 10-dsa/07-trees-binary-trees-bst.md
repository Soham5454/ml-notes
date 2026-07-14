# 07 — Trees (Binary Trees & BST)

Genuinely the biggest structural shift so far — everything up to file 06 was "flat" (linear or key-value). Trees are hierarchical, and recursion (recap file 02) is genuinely the natural tool for working with them.

## What a tree is, and basic terminology

A tree is a set of nodes connected by edges, with no cycles, starting from one **root** node. Each node can have **children**; a node with no children is a **leaf**.

```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

# Building a small tree manually:
#         5
#        / \
#       3   8
#      / \   \
#     1   4   9
root = TreeNode(5)
root.left = TreeNode(3)
root.right = TreeNode(8)
root.left.left = TreeNode(1)
root.left.right = TreeNode(4)
root.right.right = TreeNode(9)
```

This file focuses on **binary trees** — each node has AT MOST 2 children (left and right) — genuinely the most common tree type in interviews.

## Tree traversals — the fundamental operation, four ways

```python
def inorder(node, result=None):        # Left, Root, Right
    if result is None:
        result = []
    if node:
        inorder(node.left, result)
        result.append(node.value)
        inorder(node.right, result)
    return result

def preorder(node, result=None):        # Root, Left, Right
    if result is None:
        result = []
    if node:
        result.append(node.value)
        preorder(node.left, result)
        preorder(node.right, result)
    return result

def postorder(node, result=None):        # Left, Right, Root
    if result is None:
        result = []
    if node:
        postorder(node.left, result)
        postorder(node.right, result)
        result.append(node.value)
    return result

print(inorder(root))    # [1, 3, 4, 5, 8, 9]
print(preorder(root))    # [5, 3, 1, 4, 8, 9]
print(postorder(root))   # [1, 4, 3, 9, 8, 5]
```

Genuinely important, high-value fact worth memorizing: **inorder traversal of a Binary Search Tree ALWAYS produces values in sorted order** — recap the `[1, 3, 4, 5, 8, 9]` output above, already sorted, because this specific tree happens to be a valid BST. This single fact underlies a huge number of BST-related interview problems.

## Binary Search Tree (BST) — the ordering rule that makes search fast

A BST maintains a genuinely simple invariant at EVERY node: everything in the LEFT subtree is smaller, everything in the RIGHT subtree is larger.

```python
class BST:
    def __init__(self):
        self.root = None

    def insert(self, value):
        self.root = self._insert(self.root, value)

    def _insert(self, node, value):
        if node is None:
            return TreeNode(value)
        if value < node.value:
            node.left = self._insert(node.left, value)
        else:
            node.right = self._insert(node.right, value)
        return node

    def search(self, value):        # O(log n) average, O(n) worst case (unbalanced tree)
        return self._search(self.root, value)

    def _search(self, node, value):
        if node is None or node.value == value:
            return node
        if value < node.value:
            return self._search(node.left, value)
        return self._search(node.right, value)

bst = BST()
for v in [5, 3, 8, 1, 4, 9]:
    bst.insert(v)

print(bst.search(4) is not None)   # True
print(bst.search(7) is not None)   # False
```

Recap file 01's `O(log n)` category and file 12's binary search — BST search is GENUINELY the same halving idea, just applied to a tree shape instead of a sorted array: at every node, one comparison eliminates an entire subtree from consideration. The critical caveat: this `O(log n)` guarantee only holds if the tree is reasonably BALANCED — a poorly built BST (e.g. inserting already-sorted data) degenerates into a straight line, effectively becoming a linked list with O(n) search. Self-balancing trees (AVL, Red-Black) exist specifically to guarantee balance, genuinely a "know it exists" topic rather than something to implement from scratch here.

## Tree height and balance — why balance matters, concretely

```python
def tree_height(node):
    if node is None:
        return -1
    return 1 + max(tree_height(node.left), tree_height(node.right))

def is_balanced(node):
    def check(node):
        if node is None:
            return 0
        left_height = check(node.left)
        if left_height == -1:
            return -1
        right_height = check(node.right)
        if right_height == -1:
            return -1
        if abs(left_height - right_height) > 1:
            return -1
        return 1 + max(left_height, right_height)

    return check(node) != -1

print(tree_height(root), is_balanced(root))
```

Genuinely a classic interview problem in its own right — the `check()` function returning `-1` as a sentinel for "already unbalanced, stop checking" is a common, efficient pattern to avoid recomputing heights redundantly.

## Level-order traversal — where trees connect directly back to file 05's queue

```python
from collections import deque

def level_order(root):
    if not root:
        return []
    result = []
    queue = deque([root])
    while queue:
        level_size = len(queue)
        current_level = []
        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node.value)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(current_level)
    return result

print(level_order(root))   # [[5], [3, 8], [1, 4, 9]]
```

This is genuinely BFS (fully covered as a graph algorithm in file 09) applied to a tree — the `level_size` trick (snapshotting the queue's length before processing) is precisely what allows grouping nodes level-by-level, since new children get added to the SAME queue mid-iteration.

## Validating a BST — a genuinely common, subtly tricky problem

```python
def is_valid_bst(node, lower=float('-inf'), upper=float('inf')):
    if node is None:
        return True
    if not (lower < node.value < upper):
        return False
    return (is_valid_bst(node.left, lower, node.value) and
            is_valid_bst(node.right, node.value, upper))

print(is_valid_bst(root))   # True
```

Genuinely common mistake worth flagging explicitly: checking ONLY `node.left.value < node.value < node.right.value` at each node is WRONG — it misses violations further down the tree (a node three levels down on the left could still be larger than an ancestor two levels up). The bounds (`lower`, `upper`) have to propagate down through EVERY recursive call, tightening as they go — genuinely the correct, complete way to validate the BST property globally, not just locally.

## Lowest Common Ancestor (LCA) — another genuine classic

```python
def lowest_common_ancestor(root, p, q):
    if root is None or root.value == p or root.value == q:
        return root
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    if left and right:
        return root          # p and q are in DIFFERENT subtrees -- root is their LCA
    return left if left else right

lca = lowest_common_ancestor(root, 1, 4)
print(lca.value)   # 3 -- the lowest node that has both 1 and 4 as descendants
```

## Try this

```python
# Given a Binary Tree (not necessarily a BST), write a function to check if it's a
# "mirror" of itself (symmetric around its center) -- e.g.
#         1
#        / \
#       2   2
#      / \ / \
#     3  4 4  3   -- this IS symmetric
# Hint: write a helper function that checks if TWO subtrees are mirrors of each other,
# then call it on (root.left, root.right)
```

Symmetric tree checking is genuinely a favorite because it forces thinking about TWO trees simultaneously in the recursion, rather than the single-tree recursion used throughout this file — a good test of whether the recursive tree-traversal mental model has properly generalized.

## What's next
File 08 — Heaps / Priority Queues, a specialized tree structure built specifically for one job: efficiently finding the minimum or maximum element, over and over, as data changes.
