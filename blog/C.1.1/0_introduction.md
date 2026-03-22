## Introduction

### What are we building and why?

Today we are building a synthetic pretraining dataset. Specifically python programming problems and solutions designed to make a base model better at writing python code.

The inspiration comes from Section 2.3.2 of NVIDIA's Nemotron 3 Super ![technical report](https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-Super-Technical-Report.pdf). They built a dataset called Code Concepts which has 15 million Python problem solution pairs generated entirely syntheticaly. When they mixed 10B tokens of this data into the final 100B tokens of Nemotron Nano v3 pretraining HumanEval accuracy jumped 6 points (73 → 79).

In this blog, I will walk through exactly what they did, how they did it and where I think they left room for improvement. This is not a NVIDIA critique writeup, it treats their technical report as a case study to understand what we can learn from it.


### But first why target HumanEval? Isn't it saturated?

Fair question. HumanEval is 164 hand-written Python problems from 2021. Frontier models score 90%+ on it. So why bother?

The field has moved through three eras:
- Can you write a function? -> HumanEval, MBPP (2021–2023)
- Can you solve hard problems? —> LiveCodeBench, CRUXEval (2024)
- Can you be a software engineer? —> SWE-bench, Terminal Bench (2024–now)

Each era stacks on the previous one. A model that scores well on SWE-bench but poorly on HumanEval would be suspicious. The simpler benchmarks still serve as sanity checks and for smaller models they are far from saturated.

More importantly the agenda here is not to build a dataset for HumanEval. It just taking inspiration from it. HumanEval tells us what foundational Python skills look like: standalone functions, clean docstrings, edge case handling, basic algorithmic thinking. If a model can't do this, it can't do anything harder. This is Tier1 the foundation before moving to advanced concepts.

> "Benchmarks as inspiration not as targets"

Little bit on HumanEval:
HumanEval is 164 problems, all standalone pure Python functions. No imports beyond typing, no external libraries, no file I/O, no classes. Just "here's a docstring, write the function body."  


### But if goal is to learn python then why not pandas, numpy, sklearn?

If we are teaching Python, why not cover libraries? 

Four reasons:

- **Its pretraining, not SFT**. Pretraining builds foundational code reasoning: control flow, data manipulation, edge case handling, algorithmic thinking. Library specific API knowledge is better taught during supervised fine tuning.

- **Verification becomes 10x harder**. For pure algorithmic problems you can verify correctness with ast.parse. For pandas code you need actual dataframes. For matplotlib you need to render plots. 

- **Library APIs change, algorithms dont**. dp.knapsack is timeless. pd.DataFrame.groupby() might break when pandas 3.0 changes the API. Pretraining data that teaches stale APIs actively hurts the model.

- **Libraries are already in pretraining**. GitHub is full of pandas code and scikit learn tutorials. The pretraining corpus already has billions of tokens of organic library usage. What it lacked was structured, concept driven algorithmic problem solving, the exact gap this synthetic dataset fills.

