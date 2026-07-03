# 27B_NEO_CODE_IQ3_M — 03_refactor_constrained

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
import socket
import urllib.request
import urllib.error

def fetch_json(u):
    try:
        with urllib.request.urlopen(u, timeout=10) as resp:
            payload = resp.read()
    except (TimeoutError, socket.timeout) as exc:
        raise TimeoutError(f"Request to {u} timed out") from exc
    except urllib.error.HTTPError as exc:
        raise RuntimeError(f"HTTP {exc.code} error for {u}") from exc
    except urllib.error.URLError as exc:
        raise RuntimeError(f"Network error for {u}: {exc.reason}") from exc

    try:
        parsed = json.loads(payload)
    except json.JSONDecodeError as exc:
        raise ValueError(f"Invalid JSON from {u}") from exc

    if not isinstance(parsed, dict) or "data" not in parsed:
        raise KeyError("Response JSON missing 'data' key")
    return parsed["data"]
```

**Changes explained (one line each):**
- Replaced `requests` with `urllib.request` and `json` to comply with the standard-library-only constraint.
- Added an explicit 10-second timeout to prevent indefinite blocking and catch timeout failures separately.
- Split network, HTTP, and JSON parsing into distinct try/except blocks to surface caller-distinguishable exception types.
- Used exception chaining (`from exc`) to preserve original tracebacks without silencing or masking errors.
- Added explicit type/key validation before dictionary access to guarantee a clear, consistent `KeyError` for missing data.

**Explicit failure behavior:**
- **URL times out:** Raises `TimeoutError` with a descriptive message and the original exception chained.
- **Response is not JSON:** Raises `ValueError` with a descriptive message and the original `JSONDecodeError` chained.
- **JSON has no "data" key:** Raises `KeyError` with a descriptive message indicating the missing key.
