# 10 — Tries (Prefix Trees)

Genuinely a specialized tree (recap file 07) built for exactly one job: efficient string/prefix operations — autocomplete, spell-checking, word search. The last file in the Data Structures section before moving into pure Algorithms.

## What a trie actually is

A tree where each PATH from the root spells out a string, and nodes are SHARED between words with common prefixes. Genuinely the core insight: instead of storing "cat", "car", "card" as three separate strings, a trie stores the shared "ca" prefix ONCE, then branches.

```python
class TrieNode:
    def __init__(self):
        self.children = {}          # char -> TrieNode
        self.is_end_of_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True

    def search(self, word):
        node = self._find_node(word)
        return node is not None and node.is_end_of_word

    def starts_with(self, prefix):
        return self._find_node(prefix) is not None

    def _find_node(self, prefix):
        node = self.root
        for char in prefix:
            if char not in node.children:
                return None
            node = node.children[char]
        return node

trie = Trie()
for word in ["cat", "car", "card", "care", "dog"]:
    trie.insert(word)

print(trie.search("car"))         # True
print(trie.search("ca"))           # False -- "ca" was never inserted as a complete word
print(trie.starts_with("ca"))       # True -- "ca" IS a valid prefix of stored words
```

Genuinely important distinction worth being precise about: `search` checks for a COMPLETE word (needs `is_end_of_word = True`), while `starts_with` only checks that the PATH exists, regardless of whether any complete word ends exactly there.

## Why a trie beats a hash set for prefix operations

```python
# With a plain set of words, "find all words starting with 'ca'" requires checking
# EVERY word in the set -- O(n * k) where n = number of words, k = average word length

words_set = {"cat", "car", "card", "care", "dog"}
prefix_matches = [w for w in words_set if w.startswith("ca")]   # O(n*k), scans everything

# With a trie, finding the prefix node is O(k) -- just walk down k characters,
# REGARDLESS of how many total words are stored
```

This is genuinely the whole point: a hash set is excellent for exact-match lookups (recap file 06) but bad at prefix queries, since it has no concept of "shared structure" between similar strings. A trie is built specifically around that shared structure.

## Autocomplete — the canonical trie application

```python
def autocomplete(trie, prefix):
    node = trie._find_node(prefix)
    if node is None:
        return []

    results = []

    def collect_words(node, current_word):
        if node.is_end_of_word:
            results.append(current_word)
        for char, child in node.children.items():
            collect_words(child, current_word + char)

    collect_words(node, prefix)
    return results

print(autocomplete(trie, "car"))   # ['car', 'card', 'care']
```

Genuinely exactly how real autocomplete/search-suggestion features work at a basic level — walk down to the prefix node in `O(k)`, then DFS (recap file 09) through all its descendants to collect every complete word reachable from there.

## Word search in a grid — combining tries with file 09's grid-DFS pattern

```python
def find_words(board, words):
    trie = Trie()
    for word in words:
        trie.insert(word)

    rows, cols = len(board), len(board[0])
    found = set()

    def dfs(r, c, node, path):
        if (r < 0 or r >= rows or c < 0 or c >= cols or
            board[r][c] == '#' or board[r][c] not in node.children):
            return

        char = board[r][c]
        node = node.children[char]
        path += char

        if node.is_end_of_word:
            found.add(path)

        temp, board[r][c] = board[r][c], '#'    # mark visited (recap backtracking, file 02)
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
            dfs(r + dr, c + dc, node, path)
        board[r][c] = temp                        # UN-mark -- the backtrack step

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, trie.root, "")

    return found
```

Genuinely a beautiful example of THREE earlier files combining: file 09's grid-as-graph DFS pattern, file 02's choose-explore-un-choose backtracking template (marking/unmarking visited cells), and this file's trie for efficiently checking "is this partial path even a valid prefix of ANY target word" — pruning dead-end paths early instead of building every possible string and checking each against the word list separately.

## Trie space/time tradeoffs — the honest cost

```python
# Space: a trie can use MORE memory than a hash set for a small number of unrelated words
# (each character potentially becomes its own node), but becomes MORE efficient as the
# number of words with SHARED PREFIXES grows

# Time complexity summary:
# Insert:        O(k), k = word length
# Search:        O(k)
# starts_with:   O(k)
# -- genuinely independent of how many total words are stored, unlike scanning a list/set
```

## Try this

```python
# Implement a function that, given a trie already populated with a dictionary of words,
# checks if a given string can be segmented into a sequence of one or more valid
# dictionary words (e.g. "catcard" -> True, since "cat" + "card" are both valid words)
# Hint: this genuinely combines trie prefix-checking with recursion/backtracking (file 02) --
# at each position, try every valid word starting there, and recursively check the rest
```

Word Break is genuinely a classic problem that bridges tries with dynamic programming (file 15 will revisit this exact problem with a DP optimization) — solving it with plain recursion first here is a good setup for appreciating WHY the DP version becomes necessary once overlapping subproblems get large.

## Data Structures section — done. What's next
Files 03-10 covered every core data structure genuinely worth knowing for interviews: arrays/strings, linked lists, stacks/queues, hashing, trees, heaps, graphs, and tries. File 11 begins the Algorithms section — Sorting, the first "given a data structure, now DO something efficient with it" topic.
