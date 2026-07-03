# 35B_UD-Q4_K_XL — 03_refactor_constrained

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
import urllib.error
import json

def fetch_json(u):
    try:
        with urllib.request.urlopen(u, timeout=10) as resp:
            parsed = json.loads(resp.read().decode("utf-8"))
            return parsed["data"]
    except TimeoutError as e:
        raise TimeoutError(f"Request to {u} timed out") from e
    except urllib.error.HTTPError as e:
        raise RuntimeError(f"HTTP error {e.code} fetching {u}: {e.reason}") from e
    except urllib.error.URLError as e:
        raise ConnectionError(f"Network failure fetching {u}: {e.reason}") from e
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON response from {u}: {e.msg}") from e
    except KeyError:
        raise KeyError(f"Missing 'data' key in JSON response from {u}") from None
```

**Explanation of each change:**
- Moved imports to the module level to follow PEP 8 and avoid repeated module loading overhead.
- Replaced `requests` with `urllib.request` and `json` to strictly satisfy the standard-library-only constraint.
- Added a `timeout=10` parameter to prevent indefinite blocking on unresponsive or slow servers.
- Wrapped the core logic in a `try...except` block to intercept and categorize all potential failure modes.
- Added explicit exception handlers for timeouts, HTTP errors, network failures, JSON parsing errors, and missing keys.
- Used `raise ... from e` (or `from None`) to preserve or explicitly control the exception chain for accurate debugging.
- Explicitly decoded raw response bytes to UTF-8 before parsing to safely handle non-ASCII characters.

**Explicit behavior for failure cases:**
- **URL times out:** Raises a `TimeoutError` with a message indicating the specific URL timed out.
- **Response is not JSON:** Raises a `ValueError` (wrapping `json.JSONDecodeError`) detailing that the response body was not valid JSON.
- **JSON has no "data" key:** Raises a `KeyError` explicitly stating that the `data` key is absent from the parsed JSON object.
