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
import urllib.error
import json

def fetch_json(u):
    try:
        with urllib.request.urlopen(urllib.request.Request(u), timeout=10) as response:
            parsed = json.loads(response.read().decode('utf-8'))
            return parsed["data"]
    except TimeoutError as e:
        raise TimeoutError(f"Request to {u} timed out") from e
    except urllib.error.HTTPError as e:
        raise RuntimeError(f"HTTP error {e.code} for {u}: {e.reason}") from e
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON in response from {u}: {e}") from e
    except KeyError as e:
        raise KeyError(f"'data' key missing in response from {u}: {e}") from e
    except Exception as e:
        raise RuntimeError(f"Unexpected failure fetching {u}: {e}") from e
```

**Explanation of each change (one line each):**
- Replaced the third-party `requests` library with standard library `urllib.request` and `json` to satisfy the no-new-dependencies constraint.
- Added an explicit `timeout=10` argument to prevent indefinite hanging on slow or unresponsive networks.
- Wrapped the core logic in a `try/except` block to intercept failures and re-raise them as distinct, descriptive exceptions.
- Used `response.read().decode('utf-8')` to correctly convert the binary HTTP payload into a UTF-8 string before parsing.
- Maintained the exact `parsed["data"]` return statement to guarantee identical output for successful requests.
- Added a catch-all `Exception` handler to ensure no other failures are silently swallowed.

**Explicit behavior for failure cases:**
- **URL times out:** Raises `TimeoutError` with a message indicating the request timed out.
- **Response is not JSON:** Raises `ValueError` containing the underlying `JSONDecodeError` details.
- **JSON has no "data" key:** Raises `KeyError` specifying that the "data" key is missing from the parsed response.
