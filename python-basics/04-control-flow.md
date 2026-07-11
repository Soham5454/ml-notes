# 04 — Control Flow (if / elif / else)

## Basic if statement
```python
age = 20
if age >= 18:
    print("Adult")
```

## if-else
```python
age = 15
if age >= 18:
    print("Adult")
else:
    print("Minor")
```

## if-elif-else chain
```python
marks = 75

if marks >= 90:
    print("Grade A")
elif marks >= 75:
    print("Grade B")
elif marks >= 50:
    print("Grade C")
else:
    print("Fail")
```
Only ONE block runs — Python checks top to bottom and stops at the first True condition.

## Nested if
```python
num = 10
if num > 0:
    if num % 2 == 0:
        print("Positive even number")
    else:
        print("Positive odd number")
```

## Ternary operator (conditional expression)
A one-line if-else — very common in real code and interviews.
```python
age = 20
status = "Adult" if age >= 18 else "Minor"
```
Syntax: `value_if_true if condition else value_if_false`

## Truthy and Falsy values
Python treats certain values as `False` in a boolean context even if they're not literally `False`:

**Falsy values:** `False`, `0`, `0.0`, `""` (empty string), `[]` (empty list), `{}` (empty dict), `()` (empty tuple), `None`

**Everything else is truthy.**

```python
lst = []
if lst:
    print("has items")
else:
    print("empty list")   # this runs, since empty list is falsy
```
This pattern (`if lst:` instead of `if len(lst) > 0:`) is idiomatic Python — used constantly in real code.

## match-case (Python 3.10+, similar to switch-case)
```python
day = 3
match day:
    case 1:
        print("Monday")
    case 2:
        print("Tuesday")
    case 3:
        print("Wednesday")
    case _:
        print("Unknown day")   # '_' is the default/wildcard case
```

## The `pass` statement
Does nothing — used as a placeholder where syntax requires a statement but you don't want to do anything yet.
```python
if True:
    pass   # TODO: implement this later
```
