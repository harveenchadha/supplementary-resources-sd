## Stage 2: Concept Combination & Problem Generation

With a taxonomy in hand, we now generate millions of Python problems. The idea: combine concepts into sets of 1–4 and for each set, ask an LLM to generate a problem that tests all of them simultaneously.

NVIDIA's approach was straightforward — enumerate every possible combination of their 91 concepts and generate 5 problems per combo:

```
C(91,1) =          91
C(91,2) =       4,095
C(91,3) =     121,485
C(91,4) =   2,672,670
───────────────────────
Total        2,798,341 combinations
Problems     2,798,341 combinations × 5 problems each ≈ 14M problems
```

They used gpt-oss-20b a smaller, cheaper model for problem generation. Each problem had to include a descriptive function name and a problem description in the function docstring.

Note that the concepts passed into prompts use the dot notation tags from Stage 1 (algorithms.technique.two-pointer, data-structures.mapping.dictionary) not plain English phrases. The model must interpret these hierarchical labels and produce problems that meaningfully exercise the described skills.


**The Problem with Blind Enumeration at scale**

NVIDIA's brute force works because they have 91 concepts. But with our expanded 141 concept taxonomy, blind enumeration explodes:

```
C(141,1) =          141
C(141,2) =        9,870
C(141,3) =      457,310
C(141,4) =   15,777,195
─────────────────────────
Total        ~16,244,516 combinations
```

k=4 alone is ~16 million combinations which is 97% of the total. But heres the main part: **k=1, k=2 and k=3 are mostly fine.** Two concepts compose naturally ~90% of the time. Even at k=3, most triples are workable the occasional awkward one gets filtered in Stage 4. But here is a problem: most of the combinations when k=4 can be incoherent if done blindly. 


#### The Fix: Affinity Groups
The solution is **affinity groups**: clusters of concepts that naturally compose together. The 141 concepts divide into 12 groups (arrays, strings, dict/set patterns, algorithmic techniques, dynamic programming, graph algorithms, recursion/backtracking, math/number theory, data processing, Python stdlib, problem-solving strategy, security). 

Not every group pair produces coherent problems. Security + graph or recursion + data processing rarely make sense as a single problem. But arrays + techniques is bread and butter i.e. two-pointer on a sorted array, sliding window with prefix sum. 

So I have roughly defined 30 high affinity pairs, 15 triples and 8 quads that represent groups most likely to co occur in real coding problems.

The full detailed math for this step is present in Appendix C.


#### Combined k=4 total

| Source | Combos | Problems |
|---|---|---|
| Baseline (within-group) | ~16,000 | ~80,000 |
| Pattern A (2-group cross, ~30 pairs) | ~323,000 | ~1,615,000 |
| Pattern B (3-group cross, ~15 triples) | ~541,000 | ~2,705,000 |
| Pattern C (4-group cross, ~8 quads) | ~468,000 | ~2,340,000 |
| **Total k=4** | **~1,348,000** | **~6,740,000** |

Thats ~8.5% of the full C(141,4) = 15,777,195. We skip 91.5% of the combinatorial space the cross group junk that rarely produces coherent problems. {or it can but I just don't want to believe now}


#### The full dataset math
The number of problems per k is bit varied rather than keeping 5 for all.

```
k=1 (full enum, ×10):        141 combos →       1,410 problems
k=2 (full enum, ×10):      9,870 combos →      98,700 problems
k=3 (full enum, ×5):      457,310 combos →   2,286,550 problems
k=4 (affinity, ×5):     1,348,000 combos →   6,740,000 problems
──────────────────────────────────────────────────────────────────
TOTAL:                   1,815,321 combos →  9,126,660 problems
```


| | NVIDIA | Ours |
|---|---|---|
| Concepts | 91 | 141 |
| k≤3 strategy | Brute-force ×5 | Brute-force ×10 / ×10 / ×5 |
| k=4 strategy | Brute-force all 2.67M ×5 | Affinity-guided ~1.35M ×5 |
| Total problems | ~14M | ~9.1M |
| After cleaning (est.) | ~15M pairs | ~12M pairs |


Remember there is no rule that you have to invent a new approach like affinity groups at each stage, its just that if you can do something better than brute force then do it, because there is one thing that will always be in shortage: THE COMPUTE!

NVIDIA's philosophy was "generate loose, clean strict." They used a minimal prompt and relied on Stage 4 (import check + AST validation) to enforce quality. 

But we have to add constraints upfront i.e. type hints, doctests, edge cases which is a different philosophy: "generate strict, clean stricter."

Both work. NVIDIA's approach is simpler but has a higher Stage 4 rejection rate. Mine is more complex but **should** produce fewer garbage problems. 

#### Prompt 2.1: Generate Problem Statement

````
You are an expert Python problem designer. Your job is to generate a Python programming problem that tests a given set of concepts.

Requirements:
1. Write only the problem skeleton, no solution code. The skeleton is: any necessary imports, then either a function signature or a class stub with method signatures, plus a docstring.
2. Use descriptive snake_case names for functions, PascalCase for classes.
3. Include type hints for all parameters and return types.
4. The docstring must include:
   - A clear problem description
   - At least 2 examples in >>> doctest format
   - At least one edge case example (empty input, single element, etc.)
5. If the concepts require stdlib modules (heapq, collections, re, json, itertools, functools, bisect, math, dataclasses, contextlib, typing etc) include the necessary imports at the top. Only Python standard library imports are allowed.
6. Most problems should be a single function. But if the concepts involve class design, dataclasses, decorators or context managers, use the appropriate structure:
   - class-design / dataclasses → class stub with method signatures and `pass` bodies
   - decorators → a decorator function signature + a sample function it should wrap
   - context-managers → a class with `__enter__`/`__exit__` stubs or a generator with `@contextmanager`
7. For each listed concept, ask yourself: "would someone need to know this concept to solve this problem?" If the answer is no for any concept, redesign the problem until the answer is yes for all.

Wrap your output in XML tags. No explanation, no markdown fences.

<problem>
...your problem skeleton here...
</problem>

--- input below ---

<concepts>{concepts}</concepts>
````


### Step 2.2: Deduplicate generated problems

At ~9M generated problems, duplicates will be sure shot there. The same concept combination at different temperatures might produce essentially the same problem with minor wording changes. Sending duplicates to Stage 3 wastes our most expensive resource: the solution model. Remember we don't have infinite compute!

Duplication can be done at two levels:
- **Exact dedup**: Identical function names + identical docstrings → keep only one.
- **Fuzzy dedup**: Problems that differ only in variable names, wording or formatting but essentially tests the same thing.

Deduping at a level where you want to run on ~9M problems is not a very simple problem. Please refer to this [excellent resource](https://docs.nvidia.com/nemo/curator/25.09/curate-text/process-data/deduplication/index.html#text-process-data-dedup) from Nvidia on how to do fuzzy/exact dedup using Nemo Curator.

In practice, fuzzy dedup should removes 5-15% of problems. Again here writing from my experience, the number can be much higher or lower. This step is cheap (minutes on a single machine with MinHash) and saves significant cost in Stage 3. {Don't skip this}


### Step 2.3: Benchmark decontamination

At ~9M generated problems some will inevitably resemble HumanEval, MBPP because either the LLM memorized them or because common concept combinations naturally produce similar problems. If these leak into pretraining, downstream benchmark numbers are meaningless: the model isn't solving problems, it's recalling training data.

A check against a contamination set (all 164 HumanEval problems + 974 MBPP problems + any other benchmarks you plan to evaluate on) is needed.

This check can be done at two levels: exact dedup and fuzzy dedup as discussed previously. This is a cheap step as the contamination set is small, so comparing ~9M generated problems against it is minutes of work.


### Step 2.4: Concept verification and difficulty labeling

Not every generated problem actually tests the concepts it was generated from. A problem tagged with 4 concepts might only need 2. Its always a good choice to know your data upfront rather than crying later on.

**Generate as much metadata as you can to understand your data.**

#### Prompt 2.3 — Concept verification and difficulty scoring

```
You are an expert Python programming analyst.

Given a Python problem and the concepts it was designed to test, evaluate the problem on two dimensions.

Task 1 — Concept verification:
For each listed concept, determine whether a correct solution to this problem would genuinely require that concept. A concept is "required" if removing that skill from the solver's knowledge would make the problem unsolvable or significantly harder.

Task 2 — Difficulty assessment:
Rate the problem on two scales:
- Category: "easy", "medium", or "hard"
- Score: 0-10 where:
    0-2  = trivial (one-liner, basic syntax)
    3-4  = easy (straightforward application of one technique)
    5-6  = medium (requires combining techniques or careful thinking)
    7-8  = hard (requires insight, non-obvious approach, or 
            multiple algorithmic steps)
    9-10 = very hard (competition-level, requires advanced knowledge)

Return ONLY valid JSON:
{
  "verified_concepts": ["concept.actually.needed", ...],
  "dropped_concepts": {
    "concept.not.needed": "reason it's not required"
  },
  "all_concepts_tested": true/false,
  "difficulty": {
    "category": "easy|medium|hard",
    "score": 0-10,
    "reasoning": "brief explanation"
  }
}

Problem:
{problem}

Concepts it should test:
{concepts}
```

This gives us three outputs per problem:

**1. Verified concept tags.** The problem keeps only the concepts it actually tests. This metadata can be useful later on to do filtering.

**2. Concept coverage flag.** all_concepts_tested: true/false tells us whether the problem is doing its job. We don't necessarily reject problems where not all concepts are tested, but you must know your data.

**3. Difficulty labels.** The category and 0-10 score enables two things downstream:

- **Balanced dataset construction.** We can ensure the final dataset has a healthy mix of easy, medium and hard problems.
- **Curriculum-aware pretraining.** If you are planning to do phase wise pretraining this is really important. Some pretraining recipes might want to show harder problems in later phases. Difficulty labels make this possible without re-classification later.

If the distribution is too skewed lets say 80% easy, we can selectively regenerate more problems for concept combinations that consistently produce medium/hard problems or increase k=3 and k=4 problems per combo. So all this metadata is really helpful for you to focus what you are going to work on next.

#### Is this step really necessary?
If you are short on compute you can choose to skip this step, but as a good practice you should run this atleast on 10% of your data to understand the distribution of data. This was not present in NVIDIA's paper but I am sure they might be generating metadata too in similar ways.
