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
import socket

def fetch_json(u):
    try:
        with urllib.request.urlopen(u, timeout=10) as resp:
            payload = json.loads(resp.read())
            return payload["data"]
    except urllib.error.HTTPError as e:
        raise e
    except socket.timeout as e:
        raise TimeoutError("Request timed out") from e
    except urllib.error.URLError as e:
        raise e
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON: {e}") from e
    except KeyError:
        raise KeyError("Missing 'data' key") from None
```

**Explanation of each change (one line per change):**
- Replaced the third-party `requests` library with standard library modules (`urllib.request`, `json`, `socket`).
- Added a 10-second timeout to `urlopen` to prevent indefinite network blocking.
- Wrapped the core logic in a `try...except` block to intercept and handle potential failure modes.
- Re-raise `urllib.error.HTTPError` directly to preserve original HTTP status code details.
- Catch `socket.timeout` and raise a distinct `TimeoutError` to clearly identify network delays.
- Catch `urllib.error.URLError` to surface other connection/network failures.
- Catch `json.JSONDecodeError` and raise a `ValueError` to clearly signal malformed JSON.
- Catch `KeyError` and raise a descriptive `KeyError` to explicitly signal missing response fields.
- Use exception chaining (`from e`/`from None`) to preserve full traceback context for debugging.

**Explicit failure behavior:**
- If the URL times out: Raises `TimeoutError`.
- If the response is not JSON: Raises `ValueError`.
- If the JSON has no `"data"` key: Raises `KeyError`.
