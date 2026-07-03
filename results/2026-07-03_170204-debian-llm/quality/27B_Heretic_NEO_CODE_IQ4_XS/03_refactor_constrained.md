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

Here’s the refactored function:

```python
import json
import urllib.request
import urllib.error

def fetch_json(u):
    req = urllib.request.Request(u)
    try:
        with urllib.request.urlopen(req, timeout=10) as resp:
            body = resp.read().decode("utf-8")
    except (urllib.error.URLError, urllib.error.HTTPError, OSError) as e:
        raise RuntimeError(f"Network/HTTP error fetching {u}: {e}") from e

    try:
        payload = json.loads(body)
    except json.JSONDecodeError as e:
        raise ValueError(f"Response from {u} is not valid JSON: {e}") from e

    if not isinstance(payload, dict) or "data" not in payload:
        raise KeyError(f"JSON from {u} is missing the required 'data' key")

    return payload["data"]
```

One-line explanations for each change:
- Replaced `requests` with `urllib` and `json` to satisfy the standard-library-only constraint.
- Moved imports to module level for better performance and standard practice.
- Added a 10-second timeout to prevent indefinite hangs on slow or unresponsive servers.
- Wrapped HTTP/network calls in a try/except to raise a clear `RuntimeError` instead of crashing with opaque stack traces.
- Used explicit `json.loads` to catch malformed responses and raise a `ValueError`.
- Added a structure check to raise a `KeyError` when `"data"` is absent or the payload isn’t a dict.
- Preserved the original return behavior so valid inputs behave identically.
- Chained original exceptions (`from e`) so callers can inspect root causes without silent loss.

Explicit behavior for the requested cases:
- URL times out: Raises `RuntimeError` (wrapping the underlying timeout/URLError) with a clear message.
- Response is not JSON: Raises `ValueError` (wrapping `json.JSONDecodeError`) with a clear message.
- JSON has no "data" key: Raises `KeyError` with a clear message indicating the missing key.
