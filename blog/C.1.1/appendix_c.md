## Appendix C — Affinity Group Combination Math

This appendix shows the detailed calculation behind the ~1,348,000 coherent k=4 concept combinations from Stage 2. Instead of blindly enumerating all C(141, 4) = 15,777,195 four-concept sets, we define affinity groups and only generate combinations where the concepts naturally compose into a solvable problem.

### The 12 Affinity Groups

Groups map directly to the taxonomy categories in Appendix B:

| Group | Category | Size (n) | C(n, 2) |
|---|---|---|---|
| A | Arrays | 12 | 66 |
| B | Strings | 12 | 66 |
| C | Dict & Set Patterns | 9 | 36 |
| D | Algorithmic Techniques | 22 | 231 |
| E | Dynamic Programming | 10 | 45 |
| F | Graph Algorithms | 13 | 78 |
| G | Recursion & Backtracking | 6 | 15 |
| H | Math & Number Theory | 15 | 105 |
| I | Data Processing & Validation | 17 | 136 |
| J | Python Stdlib & Language | 18 | 153 |
| K | Problem-Solving Strategy | 5 | 10 |
| L | Security | 2 | 1 |

---

### Baseline: All 4 from One Group

When all 4 concepts come from the same group, coherence is guaranteed. The count is the sum of C(n_i, 4):

```
A:   495      E:   210      I: 2,380
B:   495      F:   715      J: 3,060
C:   126      G:    15      K:     5
D: 7,315      H: 1,365      L:     0
──────────────────────────────────────
                            16,181
```

Group D (Algorithmic Techniques) contributes 45% of the baseline alone — techniques like two-pointer, sliding-window and greedy are designed to combine with each other.

---

### Pattern A: 2-Group Cross (2 + 2 split)

Pick 2 concepts from group *i* and 2 from group *j*. **Formula:** C(n_i, 2) × C(n_j, 2)

We define 30 high affinity pairs — groups that a problem designer would commonly combine:

| # | Pair | Why they compose | C(n_i,2)×C(n_j,2) |
|---|---|---|---|
| 1 | D + J | technique implemented with stdlib | 35,343 |
| 2 | D + I | technique applied to data processing | 31,416 |
| 3 | D + H | mathematical algorithmic problems | 24,255 |
| 4 | I + J | stdlib tools for processing tasks | 20,808 |
| 5 | D + F | technique on graph structures | 18,018 |
| 6 | H + J | math with stdlib (math module, itertools) | 16,065 |
| 7 | A + D | technique on arrays (two-pointer, window) | 15,246 |
| 8 | B + D | technique on strings (window, stack) | 15,246 |
| 9 | H + I | mathematical data processing | 14,280 |
| 10 | F + J | graph with stdlib (deque, defaultdict) | 11,934 |
| 11 | D + E | technique meets DP (greedy vs DP) | 10,395 |
| 12 | A + J | array with stdlib (heapq, itertools) | 10,098 |
| 13 | B + J | string with stdlib (regex, Counter) | 10,098 |
| 14 | A + I | array data processing | 8,976 |
| 15 | B + I | text/string processing | 8,976 |
| 16 | C + D | dict-based techniques (two-sum, memo) | 8,316 |
| 17 | F + H | weighted graphs, distance math | 8,190 |
| 18 | A + H | mathematical ops on arrays | 6,930 |
| 19 | B + H | digit manipulation, numeric strings | 6,930 |
| 20 | E + J | DP with functools.cache | 6,885 |
| 21 | C + J | defaultdict, Counter patterns | 5,508 |
| 22 | A + F | grid/matrix as graph | 5,148 |
| 23 | E + H | combinatorial DP | 4,725 |
| 24 | C + I | data aggregation with dicts | 4,896 |
| 25 | A + B | string arrays, character matrices | 4,356 |
| 26 | D + G | recursive techniques | 3,465 |
| 27 | E + F | DP on graphs (shortest paths) | 3,510 |
| 28 | A + E | DP on arrays (subsequences) | 2,970 |
| 29 | A + C | frequency counting in arrays | 2,376 |
| 30 | G + H | recursive math (factorization) | 1,575 |
| | **Pattern A total** | | **322,954** |

Group D appears in 12 of 30 pairs because techniques are the glue — a problem rarely asks "apply two-pointer" in isolation, it asks "apply two-pointer *on an array*" or "sliding-window *with a Counter*."

---

### Pattern B: 3-Group Cross (2 + 1 + 1 split)

Pick one *primary* group to contribute 2 concepts and two *secondary* groups to contribute 1 each. **Formula:** C(n_primary, 2) × n_secondary1 × n_secondary2

The primary group anchors the core logic; secondaries add domain and implementation constraints.

| # | Primary → Secondaries | Example problem shape | Combos |
|---|---|---|---|
| 1 | D → F, J | graph algorithm implemented with deque/defaultdict | 54,054 |
| 2 | D → B, J | string technique with stdlib tools | 49,896 |
| 3 | D → A, J | array technique with stdlib tools | 49,896 |
| 4 | D → B, I | string technique with data processing | 47,124 |
| 5 | D → A, I | array technique with data processing | 47,124 |
| 6 | D → F, H | graph technique with math constraints | 45,045 |
| 7 | D → A, H | array technique with math | 41,580 |
| 8 | D → C, J | dict technique with stdlib | 37,422 |
| 9 | D → E, H | DP technique with math | 34,650 |
| 10 | J → A, I | stdlib on arrays with processing | 31,212 |
| 11 | J → B, I | stdlib on strings with processing | 31,212 |
| 12 | H → A, J | math on arrays with stdlib | 22,680 |
| 13 | H → B, J | math on strings with stdlib | 22,680 |
| 14 | I → C, H | processing with dicts and math | 18,360 |
| 15 | E → A, H | DP on arrays with math | 8,100 |
| | | **Pattern B total** | **541,035** |

D-primary dominates (9 of 15 triples) because techniques define *how* to solve the problem while the secondaries define *what* domain and *which* tools.

---

### Pattern C: 4-Group Cross (1 + 1 + 1 + 1)

One concept from each of 4 different groups. The most diverse pattern, we only pick quads where all 4 groups genuinely intersect in real problems.

**Formula:** n_i × n_j × n_k × n_l

| # | Groups | Example problem shape | Combos |
|---|---|---|---|
| 1 | A × D × I × J | array technique, processing with stdlib | 81,072 |
| 2 | B × D × I × J | string technique, processing with stdlib | 81,072 |
| 3 | A × D × H × J | array technique with math, using stdlib | 71,280 |
| 4 | B × D × H × J | string technique with math, using stdlib | 71,280 |
| 5 | A × D × F × H | array + graph technique with math | 51,480 |
| 6 | B × C × D × J | string + dict technique with stdlib | 42,768 |
| 7 | A × D × E × H | array + DP technique with math | 39,600 |
| 8 | A × C × H × J | array + dict + math with stdlib | 29,160 |
| | | **Pattern C total** | **467,712** |

---

### Grand Total

| Source | Combos | × per combo | Problems |
|---|---|---|---|
| Baseline (within-group) | 16,181 | ×5 | 80,905 |
| Pattern A (30 pairs) | 322,954 | ×5 | 1,614,770 |
| Pattern B (15 triples) | 541,035 | ×5 | 2,705,175 |
| Pattern C (8 quads) | 467,712 | ×5 | 2,338,560 |
| **Total k=4** | **1,347,882** | | **6,739,410** |

Rounded: **~1,348,000 combinations → ~6,740,000 problems.**

---

### Brute Force Comparison

```
Brute force C(141, 4) = 15,777,195
Affinity-guided         =  1,347,882
                          ──────────
Reduction               =     91.5%
```

We use 8.5% of the full combinatorial space and skip 91.5%. The skipped combinations are cross-group mixes that rarely produce coherent 4 concept problems like security + graph + recursion + DP.

### Tuning the Compatibility Matrix

The 30 pairs, 15 triples and 8 quads above are a starting point. You should tune them based on:

- **Rejection rate in Stage 4.** If a pair/triple/quad consistently fails concept verification, drop it and add a different one.
- **Difficulty distribution.** If you need more hard problems, add triples involving D + E + F (techniques + DP + graph) which tend to produce harder problems.
- **Coverage gaps.** If post-generation analysis shows underrepresented groups (like K or L), add more triples/quads involving those groups.

The framework is designed to be iterative: generate → measure → adjust compatibility → regenerate.
