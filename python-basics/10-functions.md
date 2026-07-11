# 10 — Functions

## Defining and calling a function
```python
def greet():
    print("Hello!")

greet()     # calling the function -> prints 'Hello!'
```

## Parameters and arguments
```python
def greet(name):        # 'name' is a parameter
    print(f"Hello, {name}!")

greet("Soham")            # 'Soham' is the argument passed in
```

## Return values
```python
def add(a, b):
    return a + b

result = add(3, 5)     # result = 8
```
A function without an explicit `return` returns `None` by default.

## Default parameter values
```python
def greet(name="Guest"):
    print(f"Hello, {name}!")

greet()             # Hello, Guest!
greet("Soham")        # Hello, Soham!
```
Important gotcha: never use a **mutable** default argument (like a list).
```python
def add_item(item, lst=[]):     # BAD PRACTICE
    lst.append(item)
    return lst
```
The default list is created ONCE and shared across all calls, causing unexpected accumulation. Fix:
```python
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

## Positional vs Keyword arguments
```python
def describe(name, age):
    print(f"{name} is {age} years old")

describe("Soham", 18)              # positional -> order matters
describe(age=18, name="Soham")      # keyword -> order doesn't matter
```

## *args — variable number of positional arguments
```python
def add_all(*args):
    return sum(args)

add_all(1, 2, 3, 4)     # 10 -> args is treated as a tuple: (1, 2, 3, 4)
```

## **kwargs — variable number of keyword arguments
```python
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

print_info(name="Soham", age=18)     # kwargs is treated as a dict
```

## Combining all parameter types (order matters)
```python
def func(a, b, *args, c=10, **kwargs):
    pass
# order: positional -> *args -> keyword-only with defaults -> **kwargs
```

## Scope — local vs global variables
```python
x = 10       # global variable

def modify():
    x = 20     # this creates a NEW local variable x, doesn't touch the global one
    print(x)     # 20

modify()
print(x)       # still 10 -> global x is unchanged
```

### global keyword — explicitly modify a global variable
```python
x = 10
def modify():
    global x
    x = 20     # now this DOES change the global x

modify()
print(x)       # 20
```

## Docstrings — documenting functions
```python
def add(a, b):
    """
    Adds two numbers and returns the result.
    """
    return a + b

print(add.__doc__)     # prints the docstring
```

## Functions as first-class objects
Functions can be assigned to variables, passed as arguments, and returned from other functions — this is foundational for understanding decorators and callbacks.
```python
def square(x):
    return x * x

my_func = square       # assign function to a variable (no parentheses = don't call it)
print(my_func(5))        # 25

def apply(func, value):
    return func(value)

apply(square, 4)     # 16 -> passing a function as an argument
```

## Recursion
A function calling itself — essential for trees, backtracking, divide & conquer.
```python
def factorial(n):
    if n == 0:          # base case — stops the recursion
        return 1
    return n * factorial(n - 1)     # recursive case

factorial(5)     # 120
```
Every recursive function needs a **base case**, or it recurses forever (causes `RecursionError`).
