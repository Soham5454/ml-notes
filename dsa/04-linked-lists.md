# 04 — Linked Lists

Recap file 03's array insert/delete cost note — linked lists exist specifically to fix the O(n) cost of inserting/removing at the start of an array, trading away O(1) random access to get there.

## What a linked list actually is

A sequence of **nodes**, where each node holds a value AND a reference (pointer) to the NEXT node. Unlike an array, elements aren't stored in contiguous memory — they're scattered, connected only by these pointers.

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None

    def append(self, value):        # O(n) for a singly linked list without a tail pointer
        new_node = Node(value)
        if not self.head:
            self.head = new_node
            return
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node

    def prepend(self, value):        # O(1) -- THIS is the operation arrays can't do this cheaply
        new_node = Node(value)
        new_node.next = self.head
        self.head = new_node

    def print_list(self):
        current = self.head
        values = []
        while current:
            values.append(str(current.value))
            current = current.next
        print(" -> ".join(values))

ll = LinkedList()
ll.append(1)
ll.append(2)
ll.append(3)
ll.prepend(0)
ll.print_list()   # 0 -> 1 -> 2 -> 3
```

`prepend` being genuinely O(1) here (versus an array's O(n) `insert(0, x)`, recap file 03) is the entire point of this data structure — no shifting required, just relink one pointer.

## Singly vs doubly linked lists

```python
class DoublyNode:
    def __init__(self, value):
        self.value = value
        self.next = None
        self.prev = None      # the key addition -- can traverse BACKWARD too

class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None

    def append(self, value):
        new_node = DoublyNode(value)
        if not self.head:
            self.head = self.tail = new_node
            return
        new_node.prev = self.tail
        self.tail.next = new_node
        self.tail = new_node
```

A doubly linked list trades a bit of extra memory (the `prev` pointer per node) for the ability to traverse in both directions and delete a node in O(1) GIVEN a reference to it (no need to traverse from head to find its predecessor, unlike a singly linked list).

## Reversing a linked list — genuinely one of the most classic interview questions

```python
def reverse_linked_list(head):
    prev = None
    current = head
    while current:
        next_node = current.next    # save the next node before overwriting the pointer
        current.next = prev          # reverse the pointer
        prev = current                 # move prev forward
        current = next_node            # move current forward
    return prev   # prev is now the new head

# Verifying
ll = LinkedList()
for i in [1, 2, 3, 4]:
    ll.append(i)
new_head = reverse_linked_list(ll.head)
ll.head = new_head
ll.print_list()   # 4 -> 3 -> 2 -> 1
```

`O(n)` time, `O(1)` extra space — genuinely worth being able to write this from memory, since it's asked constantly and the three-pointer (`prev`, `current`, `next_node`) juggling pattern reappears in many other linked-list problems.

## Fast & slow pointers — detecting cycles, finding the middle

```python
def has_cycle(head):
    slow, fast = head, head
    while fast and fast.next:
        slow = slow.next          # moves 1 step
        fast = fast.next.next      # moves 2 steps
        if slow == fast:            # they meet -- there's a cycle
            return True
    return False

def find_middle(head):
    slow, fast = head, head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow   # when fast reaches the end, slow is at the middle
```

This is genuinely called **Floyd's Cycle Detection** (or the "tortoise and hare" algorithm) — the slow pointer moves at half the speed of the fast one, so if there's a cycle, the fast pointer will eventually "lap" the slow one and they'll meet. If there's no cycle, `fast` simply reaches the end (`None`) first. `O(n)` time, `O(1)` space — genuinely elegant compared to the alternative of using a hash set to track visited nodes (which works but costs O(n) space).

## Merging two sorted linked lists — a genuinely common building block

```python
def merge_sorted_lists(l1, l2):
    dummy = Node(0)          # dummy node trick -- avoids special-casing the head
    tail = dummy

    while l1 and l2:
        if l1.value <= l2.value:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next
        tail = tail.next

    tail.next = l1 if l1 else l2   # attach whatever's left
    return dummy.next
```

The **dummy node trick** used here is genuinely a widely-used pattern across linked-list problems — creating a placeholder "before the real head" node avoids writing special-case logic for "what if the list is empty" or "what if we're inserting at the very start."

## Linked list vs array — the honest tradeoff table

| Operation | Array | Linked List |
|---|---|---|
| Access by index | O(1) | O(n) |
| Insert/delete at start | O(n) | O(1) |
| Insert/delete at end | O(1) amortized | O(1) with tail pointer, else O(n) |
| Insert/delete in middle | O(n) | O(n) to FIND the spot, O(1) to actually link |
| Memory | Contiguous, cache-friendly | Scattered, extra pointer overhead per node |

Genuinely important honest note: in real interview practice, linked lists are asked about constantly, but in ACTUAL production Python code, built-in lists (or `collections.deque` for queue-like needs, covered in file 05) are almost always used instead — linked lists are more of a fundamentals/interview topic than a daily-use Python tool, worth knowing that context.

## Try this

```python
# Given a singly linked list, write a function to detect if it's a palindrome
# (values read the same forwards and backwards)
# Hint: combine the find_middle technique above with the reverse_linked_list technique
# to solve this in O(n) time and O(1) extra space (not counting the list itself)
```

This is genuinely a favorite "combine two things you just learned" interview question — using fast/slow pointers to find the middle, then reversing the second half, then comparing both halves, all without extra data structures.

## What's next
File 05 — Stacks & Queues, two more fundamental structures, both genuinely easy to implement but with deep, recurring applications across the rest of this roadmap (DFS, BFS, expression evaluation, monotonic stack problems).
