# 14 — Modules & Packages

## What is a module?
A module is simply a `.py` file containing reusable Python code (functions, classes, variables) that you can import elsewhere.

## Importing modules
```python
import math
math.sqrt(16)          # 4.0

import math as m         # import with an alias (shorter name)
m.sqrt(16)

from math import sqrt      # import a specific function directly
sqrt(16)                     # no need to prefix with 'math.'

from math import sqrt, pi     # import multiple specific things

from math import *              # import everything (generally avoided — pollutes namespace, unclear where names came from)
```

## Commonly used built-in modules
```python
import math        # sqrt, floor, ceil, pi, factorial, log, etc.
import random         # randint, choice, shuffle, random
import datetime          # date and time handling
import os                   # operating system interaction (files, paths, env vars)
import sys                     # system-specific parameters (command-line args, exit)
import time                       # sleep, measuring execution time
import re                            # regular expressions (pattern matching in strings)
import collections                       # Counter, defaultdict, deque, OrderedDict
import itertools                            # permutations, combinations, product
```

### Quick examples
```python
import random
random.randint(1, 10)         # random integer between 1 and 10 (inclusive)
random.choice([1, 2, 3])        # random element from a list
random.shuffle(my_list)           # shuffles a list in place

import datetime
today = datetime.date.today()
print(today)                        # e.g. 2026-07-11

import time
start = time.time()
# ... code ...
print(time.time() - start)            # elapsed time in seconds

import itertools
list(itertools.permutations([1,2,3]))    # all orderings
list(itertools.combinations([1,2,3], 2))   # all pairs
```

## Creating your own module
If you have a file `mymath.py`:
```python
# mymath.py
def add(a, b):
    return a + b
```
You can use it in another file in the same folder:
```python
# main.py
import mymath
print(mymath.add(3, 4))     # 7
```

## Packages
A package is simply a folder containing multiple modules, with an `__init__.py` file (can be empty) marking it as a package.
```
myproject/
├── mypackage/
│   ├── __init__.py
│   ├── module1.py
│   └── module2.py
└── main.py
```
```python
from mypackage import module1
```

## `if __name__ == "__main__":`
Very common pattern — code inside this block only runs when the file is executed directly, NOT when it's imported by another file.
```python
def main():
    print("Running as main script")

if __name__ == "__main__":
    main()
```
This lets you write reusable modules that also work standalone for testing.

## pip — installing external packages
```bash
pip install numpy
pip install pandas matplotlib seaborn
pip list                  # show installed packages
pip install package --break-system-packages    # needed on some newer Linux setups
```

## Virtual environments (isolating project dependencies)
```bash
python -m venv myenv           # create a virtual environment
myenv\Scripts\activate           # activate on Windows
source myenv/bin/activate          # activate on Mac/Linux
pip install -r requirements.txt      # install all dependencies listed in a file
```
