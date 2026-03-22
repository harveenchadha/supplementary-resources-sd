## Appendix B: Improved Taxonomy (141 Concepts)

Appendix A listed all 91 raw tags from NVIDIA's dataset. This appendix shows what happens when you audit those 91m removing duplicates and filler, then filling the gaps that matter for Python pretraining.

### What Changed: Summary

| Action | Count | Details |
|---|---|---|
| **Kept from NVIDIA** | 59 | Meaningful, unique concepts retained (after merging duplicates) |
| **Removed — duplicates** | 11 | Same concept under two labels |
| **Removed — filler** | 7 | Vague or meta-tags carrying zero signal |
| **Added — new concepts** | 82 | Filling critical gaps: DP subtypes, graph algorithms, backtracking, Python stdlib & language, and more |
| **Final total** | **141** | Non overlapping, Python-native, stdlib only concepts |

---

### Part 1: What Was Removed from NVIDIA's 91

**Duplicates removed (11 tags)** — same concept appearing under multiple labels, merged into one:

algorithms.prime-sieve (= algorithms.math.prime-sieve), algorithms.strings.palindrome-check (= palindrome), algorithms.technique.bitwise-operations (= bit-manipulation), algorithms.technique.in-place-modification (= in-place), algorithms.search.linear-search (= technique.linear-search), functionality.string-processing + functionality.data-processing.string-processing (triple duplicate with algorithms.strings.processing), functionality.data-processing.text-parsing (= text-processing), functionality.security.data-encryption (= encryption), functionality.math.number-theory.modular-arithmetic (= analytics.techniques.modular-arithmetic), functionality.math.number-theory.gcd-lcm (= algorithms.math.gcd).

**Filler removed (7 tags)** — vague or meta-labels with zero testable signal:

algorithms.simple-logic, algorithms.problem.general.solving, algorithms.practice.interview, functionality.algorithmic-problem, functionality.utility.general, functionality.programming.parameters, algorithms.math.utility.

---

### Part 2: What Was Kept from NVIDIA

59 unique concepts survived after merging duplicates. These map into our new 3 level hierarchy in Part 5. The strongest NVIDIA subtrees are **data processing** (13 of 13 kept intact), **arrays** (8 kept), and **math** (9 kept). The weakest areas were **DP** (only 1 tag), **recursion** (1 tag), and **zero coverage** of trees, backtracking, binary search, greedy, or Python stdlib.

---

### Part 3: What Was Added (82 new concepts)

**Tier 1 — Critical (17):** Binary search, greedy, backtracking, prefix-sum, frequency counting (Counter), @functools.cache, custom sort keys, weighted shortest paths, topological order, connected components, cycle detection, grid-as-graph, python.collections, python.heapq, edge-case handling. These appear in 10–20% of coding problems and NVIDIA had none of them.

**Tier 2 — Important (17):** Divide-and-conquer, monotonic stack, enumerate/zip idioms, list comprehension, itertools.groupby, deduplication, combinatorial generation, split/join, character frequency, defaultdict grouping, visited-set tracker, simulation. Moderate frequency but essential for a complete taxonomy.

**Tier 3 — Selective (14):** Prefix-XOR, matrix transpose (zip(*m)), partial sort (nlargest), anagram, regex, dict inversion, tree-recursion, union-find, adjacency-dict construction, bipartite check, MST, prime factorization. Less frequent but rounds out coverage.

**Entirely new categories (zero NVIDIA coverage):** DP subtypes (9 new beyond NVIDIA's single tag), Backtracking (3), Python stdlib & language (18), Problem-solving strategy (3).

---

### Part 4: What Was Deliberately NOT Added

Competitive-programming-only concepts that don't belong in Python pretraining: linked list implementation (Python uses list/deque), segment tree / BIT (C++ specialty), suffix array/tree/automaton, network flow / bipartite matching (graduate-level), AVL / red-black tree, FFT / NTT, HLD / centroid decomposition, Mo's algorithm / sqrt decomposition, 2-SAT.

---

### Full Taxonomy (141 concepts)

All concepts use a strict 3-level hierarchy: **domain.subcategory.concept**

6 top-level domains — 4 from NVIDIA (algorithms, data-structures, functionality, analytics) + 2 new (dp, graph), organized into 12 categories.

---

#### A. Arrays (12 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | algorithms.arrays.manipulation | NVIDIA | Insert, delete, reverse, rearrange list elements |
| 2 | algorithms.arrays.processing | NVIDIA | Iterate, compute, transform over array |
| 3 | algorithms.arrays.intersection | NVIDIA | Common elements across arrays |
| 4 | algorithms.arrays.rotation | NVIDIA | Rotate by k positions |
| 5 | algorithms.arrays.rotated-array | NVIDIA | Search in rotated sorted array |
| 6 | algorithms.arrays.ranges | NVIDIA | Subarray queries, range operations |
| 7 | algorithms.arrays.max-subarray | NVIDIA | Kadane's algorithm |
| 8 | algorithms.arrays.matrix | NVIDIA | 2D array operations, grid traversal |
| 9 | algorithms.arrays.prefix-sum | NEW T1 | Prefix/suffix sum for range queries |
| 10 | algorithms.arrays.deduplication | NEW T2 | Remove duplicates, preserve order |
| 11 | algorithms.arrays.matrix-traversal | NEW T2 | Spiral, diagonal, layer-by-layer 2D traversal |
| 12 | algorithms.arrays.partitioning | NEW T2 | Partition around pivot, Dutch national flag |

#### B. Strings (12 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | algorithms.strings.manipulation | NVIDIA | Character-level modifications |
| 2 | algorithms.strings.processing | NVIDIA | General string ops |
| 3 | algorithms.strings.palindrome | NVIDIA | Palindrome detection and construction |
| 4 | algorithms.strings.prefix | NVIDIA | Common prefix, prefix matching |
| 5 | algorithms.strings.search | NVIDIA | Substring finding, pattern matching |
| 6 | algorithms.strings.sliding-window | NVIDIA | Window over characters |
| 7 | algorithms.strings.unique-characters | NVIDIA | All-unique, first non-repeating |
| 8 | algorithms.strings.parentheses-matching | NVIDIA* | Stack-based bracket matching |
| 9 | algorithms.strings.character-frequency | NEW T2 | Counter(s), frequency counting |
| 10 | algorithms.strings.anagram | NEW T3 | Anagram check and grouping |
| 11 | algorithms.strings.conversion | NEW T3 | str↔int, ord/chr, case |
| 12 | algorithms.strings.regex | NEW T3 | re.match, re.findall, re.sub |

#### C. Dict & Set Patterns (9 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | data-structures.mapping.dictionary | NVIDIA | Dict construction, lookup, iteration |
| 2 | data-structures.mapping.set-operations | NVIDIA | Union, intersection, difference |
| 3 | data-structures.mapping.frequency-counting | NEW T1 | collections.Counter pattern |
| 4 | data-structures.mapping.grouping-by-key | NEW T2 | defaultdict(list).append() |
| 5 | data-structures.mapping.caching-memoization | NEW T1 | {args: result} or @cache |
| 6 | data-structures.mapping.seen-tracker | NEW T2 | Visited set, "seen before?" |
| 7 | data-structures.mapping.key-value-inversion | NEW T3 | {v:k for k,v in d.items()} |
| 8 | data-structures.mapping.construction-from-pairs | NEW T3 | dict(zip(keys, vals)) |
| 9 | data-structures.mapping.nested-structures | NEW T2 | Nested dicts/lists, recursive access |

#### D. Algorithmic Techniques (22 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | algorithms.technique.two-pointer | NVIDIA | Opposite/same-direction pointers |
| 2 | algorithms.technique.sliding-window | NVIDIA | Fixed/variable window |
| 3 | algorithms.technique.brute-force | NVIDIA | Try all possibilities |
| 4 | algorithms.technique.in-place | NVIDIA | Modify without extra space |
| 5 | algorithms.technique.stack-usage | NVIDIA | Stack for order problems |
| 6 | algorithms.technique.memoization | NVIDIA | Cache subproblem results |
| 7 | algorithms.technique.bit-manipulation | NVIDIA | AND/OR/XOR/shifts |
| 8 | algorithms.technique.two-sum | NVIDIA | Complement lookup in dict |
| 9 | algorithms.technique.three-sum | NVIDIA | Sort + two-pointer sum |
| 10 | algorithms.technique.greedy | NEW T1 | Local → global optimal |
| 11 | algorithms.technique.interval-scheduling | NEW T1 | Sort + sweep intervals |
| 12 | algorithms.technique.backtracking | NEW T1 | Generate, prune, recurse |
| 13 | algorithms.technique.divide-and-conquer | NEW T2 | Split, solve halves, merge |
| 14 | algorithms.technique.monotonic-stack | NEW T2 | Next greater/smaller element |
| 15 | algorithms.technique.counter-tracked-window | NEW T2 | Counter-tracked window state |
| 16 | algorithms.technique.contribution-counting | NEW T2 | Per-element contribution |
| 17 | algorithms.technique.precomputation | NEW T2 | Build lookup, then query O(1) |
| 18 | algorithms.technique.prefix-xor | NEW T3 | XOR range queries |
| 19 | algorithms.technique.two-pass | NEW T2 | First pass collect info, second pass use it |
| 20 | algorithms.search.linear | NVIDIA | Sequential scan |
| 21 | algorithms.search.binary | NEW T1 | Bisect on sorted array |
| 22 | algorithms.search.binary-on-answer | NEW T1 | Parametric search |

#### E. Dynamic Programming (10 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | dp.pattern.core | NVIDIA | Overlapping subproblems, optimal substructure |
| 2 | dp.pattern.fibonacci | NEW T1 | 1D recurrence: dp[i] = f(dp[i-1]...) |
| 3 | dp.pattern.grid-pathfinding | NEW T1 | 2D grid DP, unique paths |
| 4 | dp.pattern.knapsack | NEW T1 | Subset selection with capacity constraint |
| 5 | dp.pattern.subsequence | NEW T1 | LIS, LCS, edit distance |
| 6 | dp.pattern.string-matching | NEW T1 | Regex matching, palindrome partition |
| 7 | dp.pattern.decision-making | NEW T1 | Buy/sell stock, house robber |
| 8 | dp.pattern.counting-ways | NEW T1 | Count paths, partitions, combinations |
| 9 | dp.approach.memoization | NEW T1 | @functools.cache + recursion |
| 10 | dp.approach.tabulation | NEW T1 | Iterative table filling |

#### F. Graph Algorithms (13 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | graph.traversal.general | NVIDIA | General graph traversal |
| 2 | graph.traversal.bfs | NVIDIA | BFS with deque |
| 3 | graph.traversal.dfs | NVIDIA | DFS patterns, search-based exploration |
| 4 | graph.paths.unweighted | NVIDIA | Find path between nodes |
| 5 | graph.paths.weighted | NEW T1 | Dijkstra, Bellman-Ford |
| 6 | graph.structure.connected-components | NEW T1 | BFS/DFS/DSU grouping |
| 7 | graph.structure.cycle-detection | NEW T1 | Detect cycles (directed/undirected) |
| 8 | graph.structure.union-find | NEW T3 | DSU with path compression |
| 9 | graph.structure.bipartite-check | NEW T3 | 2-coloring |
| 10 | graph.algorithms.topological-order | NEW T1 | Kahn's algorithm, dependency sort |
| 11 | graph.algorithms.minimum-spanning-tree | NEW T3 | Kruskal's / Prim's |
| 12 | graph.modeling.grid-as-graph | NEW T1 | 2D grid BFS/DFS, island counting |
| 13 | graph.modeling.adjacency-dict | NEW T2 | defaultdict(list) graph construction |

#### G. Recursion & Backtracking (6 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | algorithms.recursion.base-case | NVIDIA | Base case + recursive step |
| 2 | algorithms.recursion.divide-and-conquer | NEW T2 | Split, solve halves, merge |
| 3 | algorithms.recursion.tree-branching | NEW T3 | Multiple recursive calls (fib-style) |
| 4 | algorithms.backtracking.subsets | NEW T1 | Generate all subsets / power set |
| 5 | algorithms.backtracking.permutations | NEW T1 | Generate all orderings |
| 6 | algorithms.backtracking.constraint-solving | NEW T1 | N-queens, sudoku, word search |

#### H. Math & Number Theory (15 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | analytics.math.gcd-lcm | NVIDIA | math.gcd(), math.lcm() |
| 2 | analytics.math.primality-test | NVIDIA | is_prime check |
| 3 | analytics.math.prime-sieve | NVIDIA | Sieve of Eratosthenes |
| 4 | analytics.math.divisibility | NVIDIA | Divisibility, factors |
| 5 | analytics.math.digit-counting | NVIDIA | Digit sum, count, properties |
| 6 | analytics.math.modular-arithmetic | NVIDIA | %, pow(a,b,mod) |
| 7 | analytics.math.factorization | NEW T3 | Prime factorization |
| 8 | analytics.math.computation | NVIDIA | General math calculations |
| 9 | analytics.math.rounding | NVIDIA | floor, ceil, round |
| 10 | analytics.math.fibonacci | NVIDIA | Fibonacci variants |
| 11 | analytics.math.combinatorics | NEW T2 | math.comb(), math.perm() |
| 12 | analytics.math.binary-representation | NEW T2 | bin(), bit counting, binary string conversion |
| 13 | analytics.geometry.distance | NVIDIA* | Euclidean, Manhattan distance |
| 14 | analytics.geometry.area | NEW T3 | Area/perimeter calculations |
| 15 | analytics.statistics.descriptive | NVIDIA* | Mean, median, variance |

#### I. Data Processing & Validation (17 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | functionality.processing.conversion | NVIDIA | Type/format conversion |
| 2 | functionality.processing.reshaping | NVIDIA | Reshape, restructure |
| 3 | functionality.processing.aggregation | NVIDIA | Combine, summarize |
| 4 | functionality.processing.normalization | NVIDIA | Scale, normalize |
| 5 | functionality.processing.filtering | NVIDIA | Select by criteria |
| 6 | functionality.processing.value-search | NVIDIA | Find values in data |
| 7 | functionality.processing.parsing | NVIDIA | Parse structured input |
| 8 | functionality.processing.text-manipulation | NVIDIA | Text manipulation |
| 9 | functionality.processing.data-generation | NVIDIA | Generate test data |
| 10 | functionality.processing.split-join | NEW T2 | split(), join(), tokenization |
| 11 | functionality.processing.flatten | NEW T2 | Flatten nested lists |
| 12 | functionality.processing.groupby | NEW T2 | itertools.groupby, grouping |
| 13 | functionality.processing.json-handling | NEW T2 | json.dumps/json.loads (stdlib) |
| 14 | functionality.processing.string-formatting | NEW T2 | f-strings, .format(), template strings |
| 15 | functionality.validation.type-checking | NVIDIA | isinstance, type validation |
| 16 | functionality.validation.data-integrity | NVIDIA | Data integrity |
| 17 | functionality.validation.input | NVIDIA | Function input validation |

#### J. Python Stdlib & Language (18 concepts) — ALL NEW

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | functionality.python.collections | NEW T1 | Counter, defaultdict, deque, namedtuple |
| 2 | functionality.python.itertools | NEW T1 | product, combinations, chain, groupby |
| 3 | functionality.python.functools | NEW T1 | lru_cache, reduce, partial, cmp_to_key |
| 4 | functionality.python.heapq | NEW T1 | heappush, heappop, nlargest |
| 5 | functionality.python.bisect | NEW T2 | bisect_left, bisect_right, insort |
| 6 | functionality.python.math-module | NEW T2 | math.sqrt, math.log, math.inf, math.isclose |
| 7 | functionality.python.class-design | NEW T2 | __init__, methods, @property, dunder methods |
| 8 | functionality.python.dataclasses | NEW T2 | @dataclass, field defaults, __post_init__ |
| 9 | functionality.python.decorators | NEW T2 | Function decorators, @wraps, custom decorators |
| 10 | functionality.python.context-managers | NEW T2 | with statement, __enter__/__exit__, contextlib |
| 11 | functionality.python.generators | NEW T2 | yield, lazy evaluation, iterator protocol |
| 12 | functionality.python.exception-handling | NEW T2 | try/except/else/finally |
| 13 | functionality.python.typing-hints | NEW T3 | Type annotations, Optional, Union, List |
| 14 | functionality.python.unpacking | NEW T3 | a, *b = lst; *args, **kwargs |
| 15 | functionality.python.lambda-functional | NEW T3 | lambda, map(), filter(), reduce() |
| 16 | functionality.python.enumerate-zip | NEW T2 | enumerate(), zip() idioms |
| 17 | functionality.python.comprehension | NEW T2 | List/dict/set comprehension |
| 18 | functionality.python.combinatorial-generation | NEW T2 | itertools.product/permutations/combinations |

#### K. Problem-Solving Strategy (5 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | algorithms.strategy.duplicate-detection | NVIDIA | Find/handle/remove duplicates |
| 2 | algorithms.strategy.interval-problems | NVIDIA | Overlapping/merging intervals |
| 3 | algorithms.strategy.edge-cases | NEW T1 | Empty input, single element, negatives |
| 4 | algorithms.strategy.simulation | NEW T2 | Step-by-step simulate process |
| 5 | algorithms.strategy.space-time-tradeoff | NEW T2 | Trade memory for speed or vice versa |

#### L. Security (2 concepts)

| # | Concept | Origin | What It Should Test |
|---|---|---|---|
| 1 | functionality.security.encryption | NVIDIA | Basic encryption/decryption logic |
| 2 | functionality.security.hashing | NEW | Hash functions, checksum patterns |


---

### Final Count

| Category | Count | From NVIDIA | New |
|---|---|---|---|
| A. Arrays | 12 | 8 | 4 |
| B. Strings | 12 | 8 | 4 |
| C. Dict & Set Patterns | 9 | 2 | 7 |
| D. Algorithmic Techniques | 22 | 10 | 12 |
| E. Dynamic Programming | 10 | 1 | 9 |
| F. Graph Algorithms | 13 | 4 | 9 |
| G. Recursion & Backtracking | 6 | 1 | 5 |
| H. Math & Number Theory | 15 | 10 | 5 |
| I. Data Processing & Validation | 17 | 12 | 5 |
| J. Python Stdlib & Language | 18 | 0 | 18 |
| K. Problem-Solving Strategy | 5 | 2 | 3 |
| L. Security | 2 | 1 | 1 |
| | | | |
| **TOTAL** | **141** | **59** | **82** |
