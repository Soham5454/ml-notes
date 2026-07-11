# 12 — Exception Handling

## Why exception handling matters
Errors happen — bad input, missing files, network issues. Exception handling lets your program respond gracefully instead of crashing.

## Basic try-except
```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
```

## Catching multiple exception types
```python
try:
    num = int(input("Enter a number: "))
    result = 10 / num
except ValueError:
    print("That wasn't a valid number")
except ZeroDivisionError:
    print("Cannot divide by zero")
```

## Catching multiple exceptions in one line
```python
try:
    pass
except (ValueError, TypeError) as e:
    print(f"Error occurred: {e}")
```

## Catching any exception (use sparingly)
```python
try:
    risky_code()
except Exception as e:
    print(f"Something went wrong: {e}")
```
Catching bare `Exception` (or worse, a bare `except:`) hides bugs — prefer catching specific exceptions when you know what could go wrong.

## else clause — runs only if NO exception occurred
```python
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Error")
else:
    print(f"Success: {result}")     # runs only if try block succeeded
```

## finally clause — ALWAYS runs, error or not
```python
try:
    file = open("data.txt")
except FileNotFoundError:
    print("File not found")
finally:
    print("This always runs")     # cleanup code — closing files, connections, etc.
```

## Raising your own exceptions
```python
def set_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    return age

set_age(-5)     # raises ValueError with the custom message
```

## Common built-in exceptions
```
ValueError        -> invalid value (e.g. int("abc"))
TypeError          -> wrong type used in an operation (e.g. "2" + 2)
ZeroDivisionError    -> dividing by zero
IndexError             -> list/string index out of range
KeyError                 -> dictionary key doesn't exist
FileNotFoundError          -> file doesn't exist
AttributeError               -> object doesn't have that attribute/method
ImportError                    -> module import failed
```

## Custom exception classes
```python
class InsufficientBalanceError(Exception):
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientBalanceError("Not enough balance")
    return balance - amount

try:
    withdraw(100, 500)
except InsufficientBalanceError as e:
    print(e)     # 'Not enough balance'
```

## assert statement
Quick sanity checks during development — raises `AssertionError` if the condition is False.
```python
def divide(a, b):
    assert b != 0, "b cannot be zero"
    return a / b
```
Note: assertions can be disabled in production runs (`python -O`), so don't rely on them for real input validation — use proper `raise` for that.
