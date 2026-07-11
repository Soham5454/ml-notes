# 09 — Dictionaries

Dictionaries store **key-value pairs**. Mutable, unordered before Python 3.7, but **insertion-ordered** from Python 3.7+ onward. Extremely important for LeetCode (hashmaps) and real-world data (JSON-like structures).

## Creating dictionaries
```python
d = {"name": "Soham", "age": 18}
empty = {}
d2 = dict(name="Soham", age=18)     # alternate constructor syntax
```

## Accessing values
```python
d = {"name": "Soham", "age": 18}
d["name"]              # 'Soham'
d["city"]                # KeyError! -> key doesn't exist

d.get("city")               # None -> safe access, no error
d.get("city", "Unknown")      # 'Unknown' -> default value if key missing
```
`.get()` is almost always safer than direct indexing when the key might not exist.

## Adding / updating values
```python
d["city"] = "Bankura"     # adds new key if not present, or updates if it exists
d.update({"age": 19, "country": "India"})     # update multiple keys at once
```

## Removing values
```python
del d["city"]                  # removes key, ERRORS if key doesn't exist
d.pop("age")                     # removes and returns the value
d.pop("missing_key", None)         # safe pop with default, no error if missing
d.clear()                            # empties the dictionary
```

## Checking key existence
```python
"name" in d          # True  -> checks KEYS by default
"Soham" in d.values()  # True  -> checks VALUES explicitly
```

## Iterating over dictionaries
```python
for key in d:
    print(key)                    # iterates over keys by default

for key, value in d.items():
    print(key, value)               # iterate key-value pairs together

for value in d.values():
    print(value)                      # iterate values only

for key in d.keys():
    print(key)                          # iterate keys explicitly
```

## Dictionary comprehension
```python
squares = {x: x**2 for x in range(5)}     # {0:0, 1:1, 2:4, 3:9, 4:16}
```

## Nested dictionaries (common in real-world JSON-like data)
```python
student = {
    "name": "Soham",
    "marks": {"math": 90, "physics": 85}
}
student["marks"]["math"]     # 90
```

## Using dictionaries as counters (VERY common LeetCode pattern)
```python
word = "hello"
freq = {}
for char in word:
    freq[char] = freq.get(char, 0) + 1
print(freq)     # {'h': 1, 'e': 1, 'l': 2, 'o': 1}
```

### Better: collections.Counter (does the above automatically)
```python
from collections import Counter
freq = Counter("hello")
print(freq)     # Counter({'l': 2, 'h': 1, 'e': 1, 'o': 1})
freq.most_common(2)     # [('l', 2), ('h', 1)]  -> top 2 most frequent
```

### Also useful: collections.defaultdict
Avoids manual `.get(key, 0)` checks by auto-initializing missing keys.
```python
from collections import defaultdict
graph = defaultdict(list)
graph["A"].append("B")     # no KeyError even though "A" wasn't initialized first
```

## Dictionary keys must be immutable/hashable
```python
d = {[1,2]: "value"}     # ERROR -> list is mutable, can't be a key
d = {(1,2): "value"}      # fine -> tuple is immutable, valid key
```

## Merging dictionaries (Python 3.9+)
```python
d1 = {"a": 1}
d2 = {"b": 2}
merged = d1 | d2     # {'a': 1, 'b': 2}
```
