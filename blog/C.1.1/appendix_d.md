### Appendix D: Doctest Execution Filter — Reference Implementation

The sandboxed doctest runner used in Filter 6 (Stage 4). It executes the >>> examples in the docstring against the stitched problem+solution code and rejects any pair where the examples fail.

```
import doctest
import io
import contextlib
import signal

def run_doctests(combined_code: str, timeout: int = 10) -> bool:
    """Execute doctests in the combined problem+solution code.
    Returns True if all doctests pass."""
    
    namespace = {}
    
    try:
        signal.alarm(timeout)
        
        exec(combined_code, namespace)
        
        results = doctest.testmod(
            m=None, 
            verbose=False,
            extraglobs=namespace
        )
        
        signal.alarm(0)
        return results.failed == 0
        
    except Exception:
        signal.alarm(0)
        return False
```

**Key details:**

- signal.alarm(timeout) kills solutions that infinite-loop or hang. On Linux this works out of the box; on macOS/Windows you'd use multiprocessing with a timeout instead.
- exec(combined_code, namespace) defines the function in an isolated namespace so it doesn't pollute the global scope.
- doctest.testmod(extraglobs=namespace) runs the >>> examples using that namespace.
- Solutions involving randomness, floating-point imprecision, or system-dependent behavior (e.g., os.getpid()) can cause false failures — hence why this filter is optional and best applied to a 10-20% sample first to measure the quality delta.
