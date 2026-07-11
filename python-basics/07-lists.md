# 07 — Lists

Lists are **ordered, mutable** collections that can hold mixed data types.

## Creating lists
```python
lst = [1, 2, 3, 4]
mixed = [1, "two", 3.0, True]
empty = []
nested = [[1, 2], [3, 4]]     # list of lists (2D structure — common in LeetCode)
```

## Indexing and slicing (same rules as strings)
```python
lst = [10, 20, 30, 40, 50]
lst[0]        # 10
lst[-1]       # 50
lst[1:3]      # [20, 30]
lst[::-1]     # [50, 40, 30, 20, 10]  -> reversed
```

## Mutability — the key difference from strings/tuples
```python
lst = [1, 2, 3]
lst[0] = 100        # works fine, lists ARE mutable
print(lst)           # [100, 2, 3]
```

## Common list methods
```python
lst = [3, 1, 4, 1, 5]

lst.append(9)          # adds to end -> [3, 1, 4, 1, 5, 9]
lst.insert(0, 100)      # insert at specific index
lst.remove(1)             # removes FIRST occurrence of value 1
lst.pop()                  # removes and returns last element
lst.pop(0)                  # removes and returns element at index 0
lst.sort()                    # sorts in place, ascending
lst.sort(reverse=True)         # sorts in place, descending
lst.reverse()                    # reverses in place
lst.index(4)                       # returns index of first occurrence of 4
lst.count(1)                         # counts occurrences of value 1
lst.clear()                            # empties the list
len(lst)                                 # length of list
```

## sorted() vs .sort()
```python
lst = [3, 1, 2]
new_lst = sorted(lst)     # returns a NEW sorted list, original unchanged
lst.sort()                  # sorts the ORIGINAL list in place, returns None
```
Common mistake: `lst = lst.sort()` sets `lst` to `None` since `.sort()` returns nothing.

## Copying lists (important gotcha)
```python
a = [1, 2, 3]
b = a              # b is NOT a copy — points to the SAME list
b.append(4)
print(a)            # [1, 2, 3, 4]  -> a changed too!

c = a.copy()          # proper shallow copy
c = a[:]                # also a shallow copy (slicing the whole list)
c = list(a)               # also a shallow copy
```

## List concatenation and repetition
```python
[1, 2] + [3, 4]      # [1, 2, 3, 4]
[1, 2] * 3              # [1, 2, 1, 2, 1, 2]
```

## Checking membership
```python
3 in [1, 2, 3]      # True
```

## Iterating over lists
```python
for item in lst:
    print(item)

for i, item in enumerate(lst):     # with index
    print(i, item)
```

## List comprehensions (very Pythonic, used constantly)
```python
squares = [x**2 for x in range(5)]           # [0, 1, 4, 9, 16]
evens = [x for x in range(10) if x % 2 == 0]   # [0, 2, 4, 6, 8]
```
Full syntax: `[expression for item in iterable if condition]`
(See file 15 for deeper coverage of comprehensions.)

## 2D lists (matrices — common in LeetCode grid problems)
```python
matrix = [[0]*3 for _ in range(3)]     # 3x3 grid of zeros
# CORRECT way to create — creates independent rows

wrong = [[0]*3]*3     # WRONG — all 3 rows point to the SAME list object
wrong[0][0] = 1         # this changes ALL rows since they're the same object!
```
This is a classic Python gotcha that trips people up in LeetCode grid problems.

## Nested list access
```python
matrix = [[1, 2], [3, 4]]
matrix[0][1]     # 2   -> row 0, column 1
```

## Common built-in functions used with lists
```python
len(lst)       # length
max(lst)        # largest element
min(lst)         # smallest element
sum(lst)          # sum of all elements
```
