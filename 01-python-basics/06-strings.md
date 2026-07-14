# 06 — Strings

Strings are **immutable** sequences of characters — you can't change a character in place, only create a new string.

## Creating strings
```python
s1 = "Hello"
s2 = 'World'
s3 = """Multi-line
string"""
```

## Indexing and slicing
```python
s = "Python"
s[0]       # 'P'          -> first character
s[-1]      # 'n'          -> last character (negative indexing)
s[1:4]     # 'yth'        -> slice from index 1 to 3 (4 excluded)
s[:3]      # 'Pyt'        -> from start to index 2
s[3:]      # 'hon'        -> from index 3 to end
s[::-1]    # 'nohtyP'     -> reversed string (step = -1)
s[::2]     # 'Pto'        -> every 2nd character
```
Slicing syntax: `s[start:stop:step]` — stop is always exclusive.

## String immutability
```python
s = "hello"
s[0] = "H"        # ERROR — can't modify in place
s = "H" + s[1:]    # this is fine — creates a NEW string
```

## Common string methods
```python
s = "  Hello World  "

s.strip()          # 'Hello World'      -> removes leading/trailing whitespace
s.lower()           # '  hello world  ' -> lowercase
s.upper()           # '  HELLO WORLD  ' -> uppercase
s.replace("o", "0")  # replaces all occurrences
s.split()            # ['Hello', 'World']  -> splits by whitespace by default
s.split(",")          # splits by a specific delimiter
"-".join(["a","b","c"])  # 'a-b-c'   -> joins list into a string
s.find("World")        # index of first occurrence, -1 if not found
s.count("l")             # counts occurrences of a substring
s.startswith("Hello")     # True/False
s.endswith("World")        # True/False (careful with trailing spaces)
len(s)                       # length of string
```

## Checking string content
```python
"123".isdigit()      # True
"abc".isalpha()       # True
"abc123".isalnum()     # True
"".isspace()             # False (empty string)
"  ".isspace()            # True
```

## String formatting

### f-strings (modern, preferred — Python 3.6+)
```python
name = "Soham"
age = 18
print(f"My name is {name} and I am {age} years old")
print(f"Next year I'll be {age + 1}")   # expressions work directly inside {}
```

### .format() method (older style, still seen in codebases)
```python
print("My name is {} and I am {} years old".format(name, age))
```

### % formatting (oldest style, rarely used now)
```python
print("My name is %s and I am %d years old" % (name, age))
```

## String concatenation
```python
"Hello" + " " + "World"     # 'Hello World'
"Ha" * 3                      # 'HaHaHa'   -> repetition
```
Note: repeatedly concatenating strings in a loop is inefficient (strings are immutable, so each `+=` creates a new string object). For heavy concatenation, prefer building a list and using `"".join(list)`.

## Escape characters
```python
"Line1\nLine2"     # \n = newline
"Tab\tSpace"        # \t = tab
"She said \"Hi\""    # \" = literal double quote
"C:\\Users\\path"     # \\ = literal backslash
```

## Strings are iterable
```python
for char in "abc":
    print(char)     # a, b, c

list("abc")     # ['a', 'b', 'c']
```
