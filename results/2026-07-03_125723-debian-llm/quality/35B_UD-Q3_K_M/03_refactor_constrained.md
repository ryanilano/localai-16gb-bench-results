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
import urllib.request
import json
import urllib.error

def fetch_json(u):
    try:
        with urllib.request.urlopen(u, timeout=10) as resp:
            data = json.loads(resp.read().decode('utf-8'))
            if "data" not in data:
                raise KeyError("Response JSON lacks 'data' key") from None
            return data["data"]
    except TimeoutError as e:
        raise TimeoutError(f"Request to {u} timed out") from e
    except urllib.error.HTTPError as e:
        raise RuntimeError(f"HTTP {e.code} error for {u}") from e
    except urllib.error.URLError as e:
        raise ConnectionError(f"Network error fetching {u}") from e
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON in response: {e}") from e
```

- Moved imports to module level to follow PEP 8 and avoid redundant function-level imports.
- Replaced `requests` with `urllib.request` and `json` to satisfy the standard-library-only constraint.
- Added a `timeout` parameter to `urlopen` to prevent indefinite blocking on unresponsive servers.
- Wrapped `json.loads` in a try/except to catch malformed payloads and raise a distinct `ValueError`.
- Added an explicit `"data" in data` check to raise a clear `KeyError` instead of letting Python raise an unhandled exception.
- Categorized network and HTTP failures into `TimeoutError`, `ConnectionError`, and `RuntimeError` so callers can handle each case distinctly.
- Used exception chaining (`from e`) to preserve original tracebacks for easier debugging.

**Explicit failure behavior:**
- **URL times out:** Raises `TimeoutError` with a message indicating which URL timed out.
- **Response is not JSON:** Raises `ValueError` containing the JSON decoding error details.
- **JSON has no "data" key:** Raises `KeyError` stating that the expected key is missing from the parsed response.
