# 03 — Operators

## 1. Arithmetic Operators
```python
a, b = 7, 2

a + b     # 9   -> addition
a - b     # 5   -> subtraction
a * b     # 14  -> multiplication
a / b     # 3.5 -> division (ALWAYS returns float, even if evenly divisible)
a // b    # 3   -> floor division (integer division, rounds down)
a % b     # 1   -> modulus (remainder)
a ** b    # 49  -> exponentiation (power)
```
Important LeetCode-relevant note: `a // b` rounds **towards negative infinity**, not towards zero.
```python
-7 // 2   # -4, not -3   (this trips people up coming from C++)
```

## 2. Comparison Operators
Return a boolean (`True`/`False`).
```python
a == b    # equal to
a != b    # not equal to
a > b     # greater than
a < b     # less than
a >= b    # greater than or equal to
a <= b    # less than or equal to
```

## 3. Logical Operators
```python
True and False   # False — both must be True
True or False    # True  — at least one True
not True         # False — inverts the boolean
```
Python uses the words `and`, `or`, `not` — NOT `&&`, `||`, `!` like C++/Java.

## 4. Assignment Operators
```python
x = 5
x += 3    # same as x = x + 3   -> 8
x -= 2    # x = x - 2
x *= 2    # x = x * 2
x /= 2    # x = x / 2
x //= 2   # x = x // 2
x **= 2   # x = x ** 2
x %= 2    # x = x % 2
```

## 5. Identity Operators
Check if two variables point to the **same object in memory**, not just equal values.
```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b    # True  -> same values
a is b    # False -> different objects in memory
c = a
c is a    # True  -> c and a point to the SAME object
```

## 6. Membership Operators
Check if a value exists inside a sequence (string, list, tuple, set, dict).
```python
"a" in "apple"        # True
5 in [1, 2, 3]         # False
5 not in [1, 2, 3]     # True
```

## 7. Bitwise Operators
Operate on the binary representation of integers — common in LeetCode bit manipulation problems.
```python
a, b = 5, 3     # binary: 101 and 011

a & b     # 1   -> AND
a | b     # 7   -> OR
a ^ b     # 6   -> XOR
~a        # -6  -> NOT (inverts all bits, includes sign flip)
a << 1    # 10  -> left shift (multiply by 2)
a >> 1    # 2   -> right shift (divide by 2, floor)
```

## Operator Precedence (high to low, simplified)
```
**                    (exponent)
+x, -x, ~x            (unary)
*, /, //, %
+, -
<<, >>
&
^
|
comparisons (==, !=, <, >, etc.)
not
and
or
```
Use parentheses `()` whenever unsure — it's free and prevents bugs.
