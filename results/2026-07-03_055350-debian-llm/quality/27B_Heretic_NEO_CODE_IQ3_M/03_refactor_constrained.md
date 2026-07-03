# 27B_Heretic_NEO_CODE_IQ3_M — 03_refactor_constrained

## Prompt

Refactor this function for clarity, correctness, and proper error handling. Hard constraints: (1) keep the same function name and return type, (2) add NO new third-party dependencies (standard library only), (3) preserve the existing behavior for valid input, (4) do not swallow exceptions silently — failures must be distinguishable by the caller. Explain each change in one line.

```python
def fetch_json(u):
    import requests
    return requests.get(u).json()["data"]
```

Then state explicitly: what happens now if the URL times out, if the response is not JSON, and if the JSON has no "data" key.

## Response

Here is the refactored implementation:

```python
import json
import urllib.request
import urllib.error


class FetchJsonError(Exception):
    """Base exception for fetch_json-specific failures."""
    pass


def fetch_json(u):
    # Replaced third-party requests with stdlib urllib.request.
    # Added a timeout to prevent indefinite hangs.
    # Wrapped network/HTTP errors so callers can distinguish transport failures.
    try:
        with urllib.request.urlopen(u, timeout=10) as resp:
            body = resp.read().decode("utf-8")
    except (urllib.error.URLError, urllib.error.HTTPError) as e:
        raise FetchJsonError(f"Request failed for {u!r}: {e}") from e

    # Isolated JSON parsing so decode failures are distinguishable.
    try:
        parsed = json.loads(body)
    except json.JSONDecodeError as e:
        raise FetchJsonError(f"Invalid JSON from {u!r}: {e}") from e

    # Explicitly validated the "data" key instead of silently masking its absence.
    if not isinstance(parsed, dict) or "data" not in parsed:
        raise KeyError("'data' key missing in JSON response")

    # Preserved original return behavior for valid input.
    return parsed["data"]
```

Explicit behavior for the requested scenarios:
- **URL times out:** `urllib.request.urlopen` raises a timeout error, which is caught and re-raised as `FetchJsonError` with the original exception chained; callers can catch `FetchJsonError` (or inspect `__cause__`) to distinguish it from logic errors.
- **Response is not JSON:** `json.loads` raises `JSONDecodeError`, caught and re-raised as `FetchJsonError`; callers can distinguish this from network or missing-key failures.
- **JSON has no "data" key:** A `KeyError` is raised with a clear message; callers can catch `KeyError` specifically to handle missing-payload cases without conflating them with transport or parse errors.
