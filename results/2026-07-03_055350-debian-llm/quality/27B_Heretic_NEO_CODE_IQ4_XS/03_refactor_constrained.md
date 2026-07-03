# 27B_Heretic_NEO_CODE_IQ4_XS — 03_refactor_constrained

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
            text = resp.read().decode("utf-8")
    except urllib.error.HTTPError as e:
        raise RuntimeError(f"HTTP {e.code} for {u}") from e
    except urllib.error.URLError as e:
        raise ConnectionError(f"Request failed for {u}: {e.reason}") from e

    try:
        parsed = json.loads(text)
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON from {u}: {e}") from e

    if not isinstance(parsed, dict) or "data" not in parsed:
        raise KeyError('JSON missing required "data" key')

    return parsed["data"]
```

**Change explanations (one line each):**
- Replaced `requests` with stdlib `urllib`/`json` to satisfy “standard library only”.
- Added a 10-second timeout to prevent indefinite hangs on slow or unresponsive endpoints.
- Ordered HTTPError before URLError so callers can distinguish server status codes from transport failures.
- Isolated JSON parsing to raise a clear ValueError instead of a low-level decoder exception.
- Added an explicit key check to raise a descriptive KeyError rather than an ambiguous runtime error.
- Used exception chaining (`from e`) so callers retain full context without silent swallowing.
- Moved imports to module scope to follow Python conventions and reduce per-call overhead.

**Explicit behavior for edge cases:**
- **URL times out:** Raises `ConnectionError` with a clear message (e.g., `Request failed for ...: [Errno 110] Connection timed out`).
- **Response is not JSON:** Raises `ValueError` indicating the payload is invalid JSON.
- **JSON has no "data" key:** Raises `KeyError` stating `'JSON missing required "data" key'`.
