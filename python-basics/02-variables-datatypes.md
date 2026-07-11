# 02 — Variables & Data Types

## Variables
A variable is a name bound to a value in memory. Python is **dynamically typed** — the same variable name can be reassigned to a different type.

```python
x = 10        # x is an int
x = "hello"   # now x is a str — perfectly legal in Python
```

### Multiple assignment
```python
a, b, c = 1, 2, 3        # assign multiple variables in one line
x = y = z = 0             # assign same value to multiple variables
a, b = b, a                # swap without a temp variable (Python-specific trick)
```

## Checking type
```python
x = 5
print(type(x))    # <class 'int'>
```

## Core Data Types

### 1. Integers (`int`)
Whole numbers, positive or negative, no size limit (Python handles big integers natively — no overflow like in C++).
```python
a = 10
b = -3
big = 123456789012345678901234567890   # works fine, no overflow
```

### 2. Floating point (`float`)
Numbers with decimal points.
```python
pi = 3.14159
neg = -0.001
```
Note: floats have precision limits (floating-point rounding errors), e.g. `0.1 + 0.2` gives `0.30000000000000004`, not exactly `0.3`.

### 3. Strings (`str`)
Sequence of characters, immutable (can't be changed in place). Covered in depth in file 06.
```python
name = "Soham"
name2 = 'Also valid with single quotes'
```

### 4. Booleans (`bool`)
Only two values: `True` and `False` (capitalized). Internally, `True == 1` and `False == 0`.
```python
is_valid = True
is_empty = False
```

### 5. Complex numbers (`complex`)
Rarely needed for typical ML work, but Python supports them natively.
```python
z = 3 + 4j   # 'j' represents the imaginary unit
```

### 6. NoneType (`None`)
Represents "no value" / "nothing" — similar to `null` in other languages.
```python
result = None
```

## Type Casting (Converting between types)
```python
int("5")        # '5' -> 5        (str to int)
str(5)           # 5 -> '5'        (int to str)
float("3.14")    # '3.14' -> 3.14  (str to float)
int(3.9)         # 3.9 -> 3        (float to int, truncates — does NOT round)
bool(0)          # 0 -> False
bool(1)          # 1 -> True
bool("")         # '' -> False     (empty string is falsy)
bool("hi")       # 'hi' -> True    (non-empty string is truthy)
```

## Mutable vs Immutable types
This distinction matters a LOT in Python, including for interviews:
- **Immutable** (can't be changed after creation): `int`, `float`, `str`, `bool`, `tuple`
- **Mutable** (can be changed in place): `list`, `dict`, `set`

```python
s = "hello"
s[0] = "H"     # ERROR — strings are immutable, can't modify a character in place

lst = [1, 2, 3]
lst[0] = 100   # works fine — lists are mutable
```

## Dynamic typing gotcha
```python
x = 5
y = x       # y now points to the same int object as x
x = 10      # x is reassigned; y is untouched since int is immutable
print(y)    # still 5
```
This behaves differently for mutable types like lists — covered in file 07.
