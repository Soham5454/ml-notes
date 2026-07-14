# 17 — Bit Manipulation

A genuinely different toolkit from everything before it — operating directly on the binary representation of numbers, used for extreme space/speed optimization and forming its own recognizable category of interview questions.

## Binary representation — the foundation

```python
print(bin(13))     # '0b1101' -- 13 = 8 + 4 + 1 = 2^3 + 2^2 + 2^0
print(int('1101', 2))   # 13 -- converting back
```

Every integer is stored as a sequence of bits (0s and 1s), each position representing a power of 2 — genuinely the same base-2 positional system, just applied to how computers store numbers rather than how humans typically write them.

## The core bitwise operators

```python
a, b = 12, 10   # 1100, 1010 in binary

print(a & b)    # AND: 8  (1000) -- 1 only where BOTH bits are 1
print(a | b)    # OR:  14 (1110) -- 1 where EITHER bit is 1
print(a ^ b)    # XOR: 6  (0110) -- 1 where bits DIFFER
print(~a)        # NOT: -13 -- flips all bits (two's complement representation)
print(a << 2)    # left shift: 48 -- shifts bits left, equivalent to a * 2^2
print(a >> 2)    # right shift: 3  -- shifts bits right, equivalent to a // 2^2
```

## XOR — genuinely the most interview-relevant operator, worth understanding deeply

Two properties make XOR uniquely useful: `x ^ x = 0` (anything XORed with itself cancels to zero), and `x ^ 0 = x` (XOR with zero changes nothing).

```python
def find_single_number(arr):          # every number appears TWICE except one -- find it
    result = 0
    for num in arr:
        result ^= num                   # duplicates cancel out (x^x=0), leaving only the unique one
    return result

print(find_single_number([4, 1, 2, 1, 2]))   # 4
```

`O(n)` time, `O(1)` space — genuinely a beautiful result, impossible to achieve this efficiently without XOR's cancellation property. A hash-set-based approach (recap file 06) would also work but needs `O(n)` extra space; this needs none.

```python
def swap_without_temp(a, b):          # swapping two numbers WITHOUT a temporary variable
    a = a ^ b
    b = a ^ b        # b becomes original a
    a = a ^ b        # a becomes original b
    return a, b

print(swap_without_temp(5, 9))   # (9, 5)
```

Genuinely more of a "clever trick to know exists" than something to actually use in real code (Python's `a, b = b, a` is clearer and just as efficient) — but a common interview curiosity question.

## Checking, setting, clearing, and toggling a specific bit

```python
def get_bit(num, i):
    return (num >> i) & 1

def set_bit(num, i):
    return num | (1 << i)

def clear_bit(num, i):
    return num & ~(1 << i)

def toggle_bit(num, i):
    return num ^ (1 << i)

n = 5   # 0101
print(get_bit(n, 0))     # 1
print(set_bit(n, 1))       # 7 (0111)
print(clear_bit(n, 0))      # 4 (0100)
```

`1 << i` genuinely creates a "mask" with only the `i`-th bit set — worth internalizing this shift-to-create-a-mask pattern, since it's the building block for all four operations above.

## Counting set bits — a genuinely classic interview question, multiple approaches

```python
def count_set_bits_naive(n):
    count = 0
    while n:
        count += n & 1
        n >>= 1
    return count

def count_set_bits_brian_kernighan(n):          # genuinely faster -- skips straight to each set bit
    count = 0
    while n:
        n &= (n - 1)          # clears the LOWEST set bit each time
        count += 1
    return count

print(count_set_bits_brian_kernighan(13))   # 3 (1101 has three 1s)
```

Brian Kernighan's trick (`n & (n-1)`) is genuinely elegant: subtracting 1 from `n` flips all trailing zeros to ones and the lowest set bit to zero — ANDing with the original `n` therefore clears exactly that lowest set bit, nothing else. This means the loop runs ONCE PER SET BIT rather than once per total bit position, genuinely faster when `n` has few set bits relative to its size.

## Power of two check — a genuinely elegant one-liner

```python
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0

print(is_power_of_two(16))   # True  (10000)
print(is_power_of_two(18))    # False (10010)
```

A power of two has EXACTLY one set bit (e.g. `16 = 10000`). Using the same `n & (n-1)` trick from Brian Kernighan's method above — clearing the single set bit leaves exactly zero, which is precisely the check being performed.

## Bit manipulation for space-efficient sets — a genuinely practical application

```python
def create_bitmask_set(items, universe_size):
    mask = 0
    for item in items:
        mask |= (1 << item)
    return mask

def is_in_set(mask, item):
    return (mask >> item) & 1 == 1

s = create_bitmask_set([1, 3, 5], 8)
print(is_in_set(s, 3))   # True
print(is_in_set(s, 4))    # False
```

Genuinely a real space optimization: representing a set of up to 64 small integers (0-63) as a SINGLE integer, rather than a Python `set` object with real per-element overhead. Used in practice for DP-over-subsets problems (an extension of file 15) where each possible subset needs to be a DP state, and representing subsets as bitmasks makes both storage and set operations (union, intersection via `|`, `&`) extremely fast.

## Try this

```python
# Given an array where every element appears THREE times except one which appears once,
# find the unique element -- the simple XOR trick from this file does NOT directly work
# here (it only cancels PAIRS), so this requires tracking bit counts more carefully
# Hint: for each of the 32 bit positions, count how many numbers in the array have that
# bit set; if the count isn't divisible by 3, that bit belongs to the unique number
```

This variant is genuinely a good test of whether the XOR trick was understood as a SPECIFIC application of a more general "count bits at each position" idea, rather than memorized as an isolated trick that only applies to the exact "appears twice" scenario.

## Algorithms section — done. What's next
Files 11-17 covered the core algorithmic techniques. File 18 shifts into pure interview strategy — Pattern Recognition, mapping roughly 15 recurring patterns across everything covered in files 01-17 onto the actual shape of real interview questions.
