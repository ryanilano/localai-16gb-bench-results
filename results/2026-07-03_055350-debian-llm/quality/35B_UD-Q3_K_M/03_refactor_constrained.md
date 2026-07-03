# 35B_UD-Q3_K_M — 03_refactor_constrained

## Prompt

Refactor this function for clarity, correctness, and proper error handling. Hard constraints: (1) keep the same function name and return type, (2) add NO new third-party dependencies (standard library only), (3) preserve the existing behavior for valid input, (4) do not swallow exceptions silently — failures must be distinguishable by the caller. Explain each change in one line.

```python
def fetch_json(u):
    import requests
    return requests.get(u).json()["data"]
```

Then state explicitly: what happens now if the URL times out, if the response is not JSON, and if the JSON has no "data" key.

## Response

```python
import json
import urllib.request
import urllib.error

def fetch_json(u):
    try:
        with urllib.request.urlopen(urllib.request.Request(u), timeout=30) as resp:
            payload = json.loads(resp.read())
            if "data" not in payload:
                raise KeyError(f"Missing 'data' key: {payload}")
            return payload["data"]
    except urllib.error.HTTPError as e:
        raise RuntimeError(f"HTTP error {e.code}: {e.reason}") from e
    except urllib.error.URLError as e:
        if isinstance(e.reason, TimeoutError):
            raise TimeoutError(f"Request timed out") from e
        raise RuntimeError(f"Network error: {e.reason}") from e
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON: {e}") from e
```

**Explanation of each change (one line per change):**
- Replaced `requests` with `urllib.request` and `json` to satisfy the standard-library-only constraint.
- Added a 30-second timeout to `urlopen` to prevent indefinite blocking on unresponsive servers.
- Wrapped the core logic in a try/except block to intercept and re-raise specific exceptions with clear context.
- Added an explicit `"data"` key check to raise a descriptive `KeyError` instead of a raw lookup failure.
- Used exception chaining (`from`) to preserve original tracebacks while providing actionable error messages.

**Explicit behavior for failure cases:**
- **URL times out:** Raises a `TimeoutError` with a clear message, distinguishable from other exception types.
- **Response is not JSON:** Raises a `ValueError` wrapping the original `json.JSONDecodeError`, making it distinguishable from network or key errors.
- **JSON has no "data" key:** Raises a `KeyError` containing the full parsed payload for debugging, distinct from parsing or network failures.
