# 13 — File Handling

## Opening a file
```python
file = open("data.txt", "r")     # 'r' = read mode
content = file.read()
file.close()                       # always close to free resources
```

## File modes
```
'r'    -> read (default, errors if file doesn't exist)
'w'    -> write (creates file if missing, OVERWRITES if it exists)
'a'    -> append (creates file if missing, adds to end if it exists)
'x'    -> create (errors if file already exists)
'r+'   -> read and write
'rb'   -> read binary (for images, non-text files)
'wb'   -> write binary
```

## Preferred way: `with` statement (context manager)
Automatically closes the file even if an error occurs — this is the standard, safe way to work with files.
```python
with open("data.txt", "r") as file:
    content = file.read()
# file is automatically closed here, even if an exception happened above
```

## Reading files — different methods
```python
with open("data.txt", "r") as file:
    content = file.read()          # reads ENTIRE file as one string

with open("data.txt", "r") as file:
    lines = file.readlines()          # reads all lines into a LIST of strings

with open("data.txt", "r") as file:
    for line in file:                    # memory-efficient, reads line by line
        print(line.strip())                # .strip() removes trailing \n
```

## Writing to files
```python
with open("output.txt", "w") as file:
    file.write("Hello, World!\n")
    file.write("Second line\n")
```
Warning: `"w"` mode erases existing content the moment the file is opened.

## Appending to files
```python
with open("output.txt", "a") as file:
    file.write("This gets added to the end\n")
```

## Working with CSV files (common in ML data prep)
```python
import csv

with open("data.csv", "r") as file:
    reader = csv.reader(file)
    for row in reader:
        print(row)     # each row is a list of strings

with open("output.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["Name", "Age"])
    writer.writerow(["Soham", 18])
```

## Working with JSON files (common in ML/config data)
```python
import json

data = {"name": "Soham", "age": 18}

with open("data.json", "w") as file:
    json.dump(data, file)          # write Python dict as JSON

with open("data.json", "r") as file:
    loaded = json.load(file)         # read JSON back into a Python dict
```

## Checking if a file exists (avoid errors before opening)
```python
import os
if os.path.exists("data.txt"):
    print("File exists")
```

## Reading/writing with file paths (cross-platform safe)
```python
from pathlib import Path

path = Path("folder") / "data.txt"     # builds path correctly on any OS
if path.exists():
    content = path.read_text()
```
