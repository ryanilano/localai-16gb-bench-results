# 27B_HauhauCS_Balanced — 03_refactor_constrained

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
def fetch_json(u):
    import urllib.request
    import urllib.error
    import json

    try:
        # Use urlopen with a timeout to prevent indefinite hanging
        with urllib.request.urlopen(u, timeout=5) as response:
            # Decode bytes to string and parse JSON
            parsed = json.loads(response.read().decode('utf-8'))
            return parsed["data"]
    except urllib.error.HTTPError as e:
        # Distinguish server/client HTTP errors (4xx/5xx)
        raise RuntimeError(f"HTTP Error {e.code}: {e.reason}") from e
    except urllib.error.URLError as e:
        # Distinguish network failures and timeouts
        raise ConnectionError(f"Network/Timeout Error: {e.reason}") from e
    except json.JSONDecodeError as e:
        # Distinguish malformed JSON responses
        raise ValueError(f"Invalid JSON: {e}") from e
    except KeyError as e:
        # Distinguish missing 'data' key in the response
        raise KeyError(f"Missing 'data' key: {e}") from e
```

**Explanation of Changes:**
1.  Replaced `requests` with standard library `urllib` and `json` to satisfy the "standard library only" constraint.
2.  Added a `timeout` parameter to `urlopen` to prevent the function from hanging indefinitely on slow connections.
3.  Explicitly decoded the response bytes to a UTF-8 string before parsing to ensure correct JSON handling in Python 3.
4.  Wrapped the logic in `try/except` blocks to catch specific errors and re-raise them with context, ensuring failures are distinguishable.
5.  Handled `
