# 27B_IQ4_XS — 03_refactor_constrained

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
        with urllib.request.urlopen(u, timeout=10) as response:
            body = response.read()
    except urllib.error.URLError as e:
        if isinstance(e.reason, socket.timeout):
            raise TimeoutError(f"Request timed out: {u}") from e
        raise ConnectionError(f"Network error fetching {u}: {e}") from e

    try:
        parsed = json.loads(body)
    except json.JSONDecodeError as e:
        raise ValueError(f"Response is not valid JSON: {e}") from e

    if "data" not in parsed:
        raise KeyError("JSON response missing required 'data' key")
    return parsed["data"]
```

**One-line explanations:**
- Replaced third-party `requests` with stdlib `urllib.request` and `json` to satisfy the dependency constraint.
- Added a 10-second timeout to `urlopen` to prevent indefinite blocking on unresponsive hosts.
- Wrapped network I/O in a try/except block to explicitly raise `TimeoutError` or `ConnectionError` instead of a generic `URLError`.
- Added explicit JSON decoding with `json.JSONDecodeError` handling to raise a clear `ValueError` for malformed payloads.
- Replaced direct dictionary indexing with a key existence check to raise a descriptive `KeyError` when "data" is absent.
- Used a context manager for automatic stream closure and exception chaining (`from e`) to preserve debugging stack traces.

**Explicit failure behavior:**
- **If the URL times out:** Raises `TimeoutError` with a descriptive message.
- **If the response is not JSON:** Raises `ValueError` with a descriptive message.
- **If the JSON has no "data" key:** Raises `KeyError` with a descriptive message.
