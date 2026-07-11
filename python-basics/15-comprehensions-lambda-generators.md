# 15 — Comprehensions, Lambda, Generators, Decorators

## List Comprehensions
Compact syntax for creating lists — Pythonic replacement for many for-loops.
```python
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
pairs = [(x, y) for x in range(3) for y in range(3)]     # nested loops in one line
```
Equivalent long-form:
```python
squares = []
for x in range(10):
    squares.append(x**2)
```

## Set Comprehensions
```python
unique_lengths = {len(word) for word in ["hi", "hello", "hey"]}     # {2, 5, 3}
```

## Dictionary Comprehensions
```python
squares_dict = {x: x**2 for x in range(5)}          # {0:0, 1:1, 2:4, 3:9, 4:16}
inverted = {v: k for k, v in original_dict.items()}    # swap keys and values
```

## Nested comprehension (careful — readability trade-off)
```python
matrix = [[1, 2], [3, 4]]
flattened = [num for row in matrix for num in row]     # [1, 2, 3, 4]
```

## Lambda Functions
Anonymous, one-line functions — used when you need a quick throwaway function, often as an argument to another function.
```python
square = lambda x: x ** 2
square(5)     # 25

add = lambda a, b: a + b
add(3, 4)       # 7
```
Equivalent to:
```python
def square(x):
    return x ** 2
```

### Common use: sorting with a custom key
```python
students = [("Soham", 90), ("Alex", 85)]
students.sort(key=lambda x: x[1])          # sort by second element (marks)

words = ["banana", "kiwi", "apple"]
words.sort(key=lambda w: len(w))              # sort by string length
```

## map(), filter(), reduce()
```python
nums = [1, 2, 3, 4]

list(map(lambda x: x**2, nums))              # [1, 4, 9, 16] -> apply function to each item
list(filter(lambda x: x % 2 == 0, nums))        # [2, 4]         -> keep items matching condition

from functools import reduce
reduce(lambda a, b: a + b, nums)                   # 10             -> cumulatively combine items
```
Note: list comprehensions are often preferred over `map`/`filter` in modern Python for readability, but they show up often in older code and interviews.

## Generators
Functions that yield values one at a time, pausing state between calls — memory-efficient for large sequences since they don't store everything in memory at once.
```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i          # pauses here, resumes on next call
        i += 1

for num in count_up_to(5):
    print(num)     # 1, 2, 3, 4, 5
```
Difference from `return`: `return` exits the function completely; `yield` pauses it, remembering its state for the next call.

### Generator expressions (like list comprehensions, but lazy)
```python
gen = (x**2 for x in range(1000000))     # doesn't compute all values immediately
next(gen)                                    # 0 -> computes one value at a time
```
Use `()` instead of `[]` — this avoids building a huge list in memory upfront.

## Decorators
Functions that wrap another function to extend its behavior without modifying its code directly.
```python
def my_decorator(func):
    def wrapper():
        print("Before the function runs")
        func()
        print("After the function runs")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# Before the function runs
# Hello!
# After the function runs
```
`@my_decorator` is shorthand for `say_hello = my_decorator(say_hello)`.

### Practical example — timing a function
```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"Took {time.time() - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

slow_function()     # Took 1.0001 seconds
```
