# 27B_Heretic_Youssofal_Q3_K_M — 03_refactor_constrained

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

def fetch_json(u):
    with urllib.request.urlopen(u, timeout=10) as response:
        data = json.loads(response.read())
        if "data" not in data:
            raise KeyError("Response JSON lacks required 'data' key")
        return data["data"]
```

**Explanation of changes:**
- Removed `requests` dependency and replaced it with standard library `urllib` and `json` modules.
- Added a 10-second request timeout to prevent indefinite blocking on unresponsive endpoints.
- Added explicit validation for the missing "data" key to fail fast with a clear, actionable error.
- Used a context manager to guarantee safe closure of network resources regardless of success or failure.
- Removed all exception suppression to ensure every failure propagates clearly and remains distinguishable.

**Explicit behavior on failure cases:**
- If the URL times out: Raises `TimeoutError` (or `urllib.error.URLError` wrapping it), which is distinguishable and not swallowed.
- If the response is not JSON: Raises `json.JSONDecodeError`, which is distinguishable and not swallowed.
- If the JSON has no "data" key: Raises `KeyError` with a clear message, which is distinguishable and not swallowed.
