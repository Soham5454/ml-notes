# 01 — Introduction to Python

## What is Python?
Python is a high-level, interpreted, general-purpose programming language. "High-level" means it abstracts away memory management and hardware details. "Interpreted" means code runs line-by-line through an interpreter (CPython, the standard one) instead of being compiled to machine code beforehand like C++.

## Why Python is popular for ML/AI
- Simple, readable syntax — close to plain English, less boilerplate than C++/Java
- Massive ecosystem: NumPy, pandas, PyTorch, scikit-learn, etc. are all Python-first
- Huge community, so almost every error has a documented solution
- Great as "glue code" — easy to call C/C++ libraries under the hood for speed while writing Python on top

## Interpreted vs Compiled (why Python is "slower")
- Compiled languages (C++) convert code to machine code once, then run — fast execution
- Python interprets code line by line at runtime — flexible, easy to debug, but slower raw execution
- This is why performance-critical ML libraries (NumPy, PyTorch) are written in C/C++ under the hood, with Python as the interface

## Installing & running Python
- Check version: `python --version` or `python3 --version`
- Run a script: `python filename.py`
- Interactive shell (REPL): just type `python` in terminal — good for quick testing

## Writing your first program
```python
print("Hello, World!")
```
`print()` is a built-in function that outputs text to the console.

## Comments
```python
# This is a single-line comment

"""
This is a multi-line comment / docstring
Often used to document what a function or file does
"""
```

## Indentation matters
Unlike C++/Java (which use `{ }`), Python uses **indentation** to define code blocks. This is not optional — incorrect indentation causes errors.

```python
if True:
    print("This is inside the if block")   # 4 spaces indent (standard)
print("This is outside the if block")
```

## Variables — no type declaration needed
```python
x = 5          # Python figures out this is an integer
name = "Soham" # this is a string
```
This is called **dynamic typing** — you don't declare `int x = 5;` like in C++. The interpreter infers the type at runtime.

## Case sensitivity
`Name`, `name`, and `NAME` are three different variables in Python.

## Keywords (reserved words — can't be used as variable names)
```
False, None, True, and, as, assert, async, await, break, class, continue,
def, del, elif, else, except, finally, for, from, global, if, import,
in, is, lambda, nonlocal, not, or, pass, raise, return, try, while,
with, yield
```
