# 05 — Loops

## for loop
Iterates over a sequence (list, string, tuple, range, dict, etc.) — Python's for loop is more like a "for-each" than C++'s counting for loop.

```python
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for fruit in ["apple", "banana", "cherry"]:
    print(fruit)

for char in "hello":
    print(char)
```

## range() function
```python
range(5)          # 0 to 4
range(2, 8)        # 2 to 7  (start, stop)
range(0, 10, 2)     # 0, 2, 4, 6, 8   (start, stop, step)
range(10, 0, -1)    # 10 down to 1   (negative step = countdown)
```
`range()` is memory-efficient — it generates numbers on the fly instead of storing a full list.

## while loop
Runs as long as a condition is True.
```python
count = 0
while count < 5:
    print(count)
    count += 1
```
Careful: forgetting to update the loop variable causes an **infinite loop**.

## break
Exits the loop immediately.
```python
for i in range(10):
    if i == 5:
        break
    print(i)     # prints 0, 1, 2, 3, 4 then stops
```

## continue
Skips the rest of the current iteration, moves to the next one.
```python
for i in range(5):
    if i == 2:
        continue
    print(i)     # prints 0, 1, 3, 4 (skips 2)
```

## else clause on loops (Python-specific, uncommon elsewhere)
The `else` block runs only if the loop completes WITHOUT hitting a `break`.
```python
for i in range(5):
    if i == 10:
        break
else:
    print("Loop completed without break")   # this runs, since break never triggered
```
Useful pattern: searching for something and doing a fallback action only if not found.

## Nested loops
```python
for i in range(3):
    for j in range(3):
        print(i, j)
```

## enumerate() — get index + value together
```python
fruits = ["apple", "banana", "cherry"]
for index, fruit in enumerate(fruits):
    print(index, fruit)
# 0 apple
# 1 banana
# 2 cherry
```
Much cleaner than manually tracking `i = 0` and incrementing.

## zip() — loop over multiple sequences together
```python
names = ["Soham", "Alex"]
ages = [18, 20]
for name, age in zip(names, ages):
    print(name, age)
# Soham 18
# Alex 20
```

## Loop control summary (interview-relevant)
- `break` → exit loop entirely
- `continue` → skip current iteration, go to next
- `pass` → do nothing, placeholder (does NOT skip or exit)
