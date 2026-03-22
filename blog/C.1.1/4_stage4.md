## Stage 4: Solution Cleaning & Validation

This is where ~36.4M raw pairs become a clean final dataset. Every problem solution pair passes through a pipeline of filters: each catching a different class of failure [this is such a headache]. A pair must survive all filters to make it into the final dataset.

NVIDIA applied three filters (import check, solution extraction, AST validation) and saw a ~35% rejection rate. These three are a good starting point but I would suggest four more which are more complicated than AST validation as it alone can't detect major issues.

### Filter 1: Import validation

The solution must not add any imports beyond what the original problem specifies.

```python
# Problem says:
from typing import List
import heapq

# Solution adds:
import numpy as np    # ← REJECT: unauthorized import
```

Why this matters: without this check, the solution model cheats by importing libraries that do the heavy lifting. A solution that imports 'networkx' for a graph problem hasn't demonstrated the algorithmic thinking the concepts were supposed to test.

Extract imports from both the problem and solution using ast.parse(), compare the sets and reject any solution where solution_imports - problem_imports is non empty. Simple solution no LLM calls!


### Filter 2: Signature and docstring validation

The solution model outputs complete code inside <solution> tags. Extract that code (simple regex) and then verify it preserved the original problem skeleton: same function/class names, same parameter names, same docstring.

```python
import ast, re

def extract_solution(raw_output: str) -> str | None:
    match = re.search(r'<solution>(.*?)</solution>', raw_output, re.DOTALL)
    return match.group(1).strip() if match else None

def signatures_match(problem_code: str, solution_code: str) -> bool:
    """Check that the solution kept names, params, and docstring intact."""
    prob_tree = ast.parse(problem_code)
    sol_tree = ast.parse(solution_code)

    for prob_node, sol_node in zip(
        (n for n in ast.walk(prob_tree) if isinstance(n, (ast.FunctionDef, ast.ClassDef))),
        (n for n in ast.walk(sol_tree) if isinstance(n, (ast.FunctionDef, ast.ClassDef)))
    ):
        if prob_node.name != sol_node.name:
            return False
        if isinstance(prob_node, ast.FunctionDef):
            prob_params = [a.arg for a in prob_node.args.args]
            sol_params = [a.arg for a in sol_node.args.args]
            if prob_params != sol_params:
                return False
    return True
```

If extract_solution returns None → model didn't use XML tags, reject. 
If signatures_match returns False → model renamed something, reject. 

No stitching, no subtle bugs. Either accept the solution exactly as the model wrote it or throw it away. We don't need to keep things where LLMs didn't follow our instructions.

NVIDIA's pipeline needed stitching because their prompt didn't anchor names strongly. Ours explicitly instructs the model to keep all names and parameters intact and uses XML output tags for clean extraction, so compliance is high enough that rejecting non compliant outputs is cheaper than trying to surgically repair them.


### Filter 3: AST validation

The extracted solution code must parse as valid Python.

```python
import ast

try:
    ast.parse(solution_code)
except SyntaxError:
    # REJECT: not valid Python
```

This catches unclosed brackets, malformed strings, indentation errors, incomplete function bodies and any other syntax issues. Its fast, deterministic and requires no execution environment.


### Filter 4: Line limit enforcement

The solution model was instructed to stay under the line limit (40/60/80/100 by k), but LLMs count poorly. Enforce it programmatically:

```python
def count_code_lines(solution_body: str) -> int:
    """Count non-empty, non-comment lines."""
    return sum(
        1 for line in solution_body.strip().split('\n')
        if line.strip() and not line.strip().startswith('#')
    )
```

Solutions exceeding the limit for their k level are discarded. This ensures the pretraining data consistently models concise solutions for simple problems and allows more room only where complexity demands it.


### Filter 5: Trivial and broken solution detection

AST validation confirms the code is syntactically valid — but syntactically valid code can still be useless:

```python
# Passes AST check but is obviously wrong:
def count_primes(n: int) -> int:
    """Count primes less than n."""
    pass

def count_primes(n: int) -> int:
    """Count primes less than n."""
    return None

def count_primes(n: int) -> int:
    """Count primes less than n."""
    return 0

def count_primes(n: int) -> int:
    """Count primes less than n."""
    ...
```

Reject solutions where the entire function body (or all method bodies in a class) is one of: pass, ..., return None, return 0, return "", return [], return {} or return False. Also reject solutions shorter than 2 non trivial lines, a real solution to a real problem is never a single return statement.

Remember the order of filters is very important. Like run this filter 5 only after executing the previous filters.

### Filter 6: Doctest execution (optional)

This is the most powerful filter and the one NVIDIA didn't use (or they did but never told). If the problem includes >>> doctest examples in the docstring, we can actually run them against the solution using pythons built in doctest module inside a sandboxed environment with a timeout. Read the line again, inside a sandboxed environment. Sandboxed environment is a must.

Why this is powerful: AST validation confirms the code is *valid Python*. Doctest execution confirms it *actually works*. A solution that parses correctly but produces wrong outputs gets caught here.

Why it's optional: it requires an execution environment (sandboxed, with timeouts and memory limits). Some solutions will involve randomness, floating point imprecision, or system dependent behavior that causes false failures. And at ~36M pairs, execution takes real compute time.

of course, if you have compute constraints you can run doctest execution on a sample (10-20% of pairs) to measure the quality delta. If it catches a significant additional percentage beyond the other filters, make it a standard filter. If most of what it catches is floating point edge cases, keep it optional.

The reference implementation for the sandboxed doctest runner is in Appendix D.


### Filter 7: Duplicate detection

At ~36M pairs, some concept combinations will produce near identical problems and some solutions will be near identical across problems. Excessive duplication wastes pretraining tokens.

Deduplicate at two levels like we did previously:

- **Exact dedup:** same problem text + same solution body → keep only one. This is a simple hash comparison after stripping whitespace.
- **Fuzzy dedup:** solutions that differ only in variable names, whitespace, or comment phrasing → keep only one. The same approach discussed in Stage 2 for problem level dedup. (Will write a complete writeup on this one if there is more interest) 
