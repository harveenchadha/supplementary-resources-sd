## Stage 1: Build a Concept Taxonomy

### Purpose

The main purpose of this stage is to build a controlled taxonomy of python programming concepts that will seed all the downstream data generation. The taxonomy acts as the diversity engine for the entire pipeline.

So in easy words: Given a python problem, what are the concepts that problem is testing??

The approach has three steps: 
- build a broad taxonomy from curated sources
- normalize it into a consistent hierarchy
- filter it through coding benchmarks to keep only what matters for Python problem-solving.

### Step 1.1: Extract concepts from curated sources
One straightforward way to build the initial taxonomy is to run an LLM as a topic classifier on Python and DSA textbooks. Honestly, you don't even have to run on the entire book, only detailed outline and table of contents is enough.

I used 6-7 books to get initial taxonomy. Books like Data Structures and Algorithms using Python, Fluent Python, Python Cookbook, Effective Python and problem tags from LeetCode and Codeforces are a very good starting point.

Just passed the table of contents chapter by chapter and got like 900 raw concepts. These concepts use a hierarchical dot-notation format. Here are some real examples from the dataset:

- algorithms.technique.two-pointer
- algorithms.technique.sliding-window
- analytics.math.gcd-lcm
- data-structures.mapping.dictionary
- functionality.processing.reshaping

Think of these as atomic concepts that can be combined. A problem tagged algorithms.technique.two-pointer + algorithms.arrays.processing might ask you to find pairs in a sorted array. I hope you got this part!!


#### Alternate approach

NVIDIA started with extraction of thousands of concepts from their pretraining code corpus and then filtered them through HumanEval. They built the big taxonomy first, then used it as a lens to classify HumanEval.

Think of it like this: they had a dictionary of 3,000 programming concepts. They looked at each HumanEval problem and asked which concepts from our dictionary does this problem test? 91 dictionary entries matched. They used those 91.

Their focus was mainly on "Which programming concepts does HumanEval actually test?". So after extracting concepts from all 164 HumanEval problems and filtering pretraining code corpus based on concepts, they landed on 91 unique concepts. After auditing those 91, found ~11 duplicates and ~7 filler tags, leaving ~73 meaningful concepts. Suggesting an improved taxonomy (see Appendix B) that extends this to 141 by filling critical gaps like binary search, greedy, DP subtypes, graph algorithms, and Python stdlib.


#### PROMPT 1.1 (Concept Extraction):

```
You are a programming education expert. Analyze this reference material and extract every distinct programming concept, technique, data structure, and pattern it teaches or covers.

Return a taxonomical dot-notation representation of each concept, organized hierarchically by category. Each concept should follow the pattern: category.subcategory.specific-concept

Rules:
- Each concept should be a reusable skill, not specific to this problem
- Use lowercase-with-hyphens within levels, dots between levels
- Include 1-5 concepts, ranked by importance

Examples of concept labels:
- algorithms.technique.two-pointer
- data-structures.mapping.dictionary
- functionality.processing.parsing
- algorithms.technique.bit-manipulation

Return ONLY valid JSON:
{
  "concepts": ["category.subcategory.detail", ...]
}

Material:
{content}
```

Note: For using NVIDIA's approach you can use the same prompt with minimal modifications where you classify pretraining code samples instead of reference material.

**Suggested Model: Qwen-3.5-35B-A3B**

Because Prompt 1.1 is not constrained, you get a richer initial concept space. The mess is intentional. We need to write another prompt to clean this up which is basically merging related concepts and deduping them.


### Step 1.2: Normalize into a clean taxonomy
The raw concepts from multiple books and sources will have duplicates, synonyms and inconsistent naming. Binary search might appear as algorithms.searching.binary-search or algorithms.technique.binary-search and data-structures.arrays.binary-search across different sources.

I used the Prompt 1.2 to normalize everything into a consistent 3 level hierarchy.

#### PROMPT 1.2 (Dedup/Clean Taxonomies):

```
You are a taxonomy designer.

Below are {n} programming concept tags that were independently extracted from Python textbooks and DSA references. There are many duplicates, synonyms and inconsistent names.

Your job: produce a CLEAN, CANONICAL taxonomy.

Step 1 — Merge synonyms:
    - algorithms.pointers.two-pointer and patterns.array.two-pointers → algorithms.technique.two-pointer  
    - data-structures.hash.hashmap and mapping.dict.dictionary → data-structures.mapping.dictionary
    - math.arithmetic.modular and algorithms.math.modular-arithmetic → analytics.math.modular-arithmetic

Step 2 — Remove noise:
    - Too vague: algorithms.general.problem-solving, functionality.utility.general
    - Too specific to one problem: algorithms.array.leetcode-121-stock-profit
    - About syntax, not concepts: basics.control-flow.for-loop, "basics.syntax.if-statement
    - About non-programming concerns: practice.interview.preparation

Step 3 — Fix hierarchy depth:
Every tag that survived Steps 1-2 must be exactly 3 levels.

2-level tag → infer and add the missing subcategory:
    - algorithms.recursion → algorithms.recursion.base-case
    - data-structures.stack → algorithms.technique.stack-usage

4-level tag → flatten by merging the two most related levels:
    - algorithms.graph.traversal.bfs → graph.traversal.bfs
    - functionality.data.processing.filtering → functionality.processing.filtering


Step 4 — Standardize naming conventions:
    - All lowercase
    - Hyphens between words within a level: two-pointer not two_pointer or twopointer
    - Dots between levels: algorithms.technique.two-pointer
    - No trailing hyphens or dots
    - Consistent vocabulary:
        "dictionary" not "dict" or "hashmap" or "hash-table" (pick one, use everywhere)
        "two-pointer" not "two-pointers" or "2-pointer"
    - Consistent top-level domains: algorithms, data-structures, functionality, analytics, dp, graph

Output JSON format:
{{
  "taxonomy": ["canonical.concept.one", ...],
  "merge_log": {{"kept": ["merged_synonym_1", "merged_synonym_2"]}},
  "removed": {{"tag": "reason"}},
  "category_summary": {{"category_name": count}}
}}

Raw tags:
{tags}

```


### Step 1.3: Filter through coding benchmarks 

This is an optional step but its better to keep your concept generations on target. Not every programming concept in a textbook might be useful for our dataset. Django middleware patterns, async/await paradigms and deployment scripts are real skills but they are not testable as standalone function completion problems, which is what we are generating. (Always remembers the constraints we have while taking a decision)


#### Prompt 1.3 — Benchmark classification (constrained):

```
You are an expert Python programming analyst.

Given a Python problem and a taxonomy of programming concepts, identify which concepts from the taxonomy are needed to solve this problem.

IMPORTANT: Choose ONLY from the provided taxonomy. Do not invent new concepts. If a problem requires a skill not in the taxonomy, use the closest match.

Taxonomy:
{taxonomy}

Problem:
{problem}

Return ONLY valid JSON:
{
  "task_id": "{task_id}",
  "concepts": ["concept.from.taxonomy", ...],
  "reasoning": "brief justification"
}
```

Any concept that appears in at least once survives into the final taxonomy. Concepts that exist only in the textbook vocabulary but never match a benchmark problem get dropped they're valid programming knowledge but not useful for this dataset.

**Frankly I will just handpick rather than giving every job to LLM**

### Are these taxonomies enough??

NVIDIA’s 91 concepts were filtered through one lens: what does HumanEval test? Since HumanEval allows no imports beyond typing, no file I/O and no classes, their taxonomy naturally excluded anything that touches Python’s standard library. No collections.Counter, no heapq, no itertools.

This was the right decision for their goal, they wanted to prove that concept driven synthetic data improves a specific benchmark and they did (+6 points on HumanEval).

But our goal is different. We don’t want to optimize for one benchmark, we are building foundational Python fluency for pretraining. The dataset should teach the model how a competent Python developer actually thinks and writes code.

So I did some manual analyis and built an improved taxonomy. I divide these into tiers:

*Tier 1: Critical gaps (17 concepts):* Binary search, greedy algorithms, backtracking, DP subtypes, graph specificity (Dijkstra, topological sort, connected components, cycle detection), prefix sum, Python stdlib patterns (collections.Counter, heapq, itertools, functools.cache).

*Tier 2: Important additions (17 concepts):* Divide and conquer, list/dict comprehension, enumerate, zip, idioms, character frequency, combinatorics, monotonic stack, sliding window with counter, simulation, precomputation, generator patterns, exception handling, basic class design, two-pass technique, json handling.

*Tier 3 — Selective cherry picks (14 concepts):* Union find, geometry distance, number factorization, anagram patterns, regex, string conversion, array flattening, key value inversion, bisect module, decorators, context managers.

There were some duplicates and zero signal concepts in Nvidia’s taxonomy (Appendix A) so after removing them I was left with 73 concepts. After combining 73 concepts with the ones mentioned above I got: 141 unique, non-overlapping, Python-native concepts across 12 categories. The full taxonomy is in Appendix B.

