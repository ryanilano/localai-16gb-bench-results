# 27B_HauhauCS_Balanced_Q3_K_P — 03_refactor_constrained

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
        with urllib.request.urlopen(u, timeout=10) as response:
            return json.loads(response.read())["data"]
    except urllib.error.HTTPError as e:
        raise ValueError(f"HTTP {e.code} error from {u}") from e
    except urllib.error.URLError as e:
        raise RuntimeError(f"Network or timeout error: {e.reason}") from e
    except json.JSONDecodeError as e:
        raise ValueError(f"Response is not valid JSON: {e}") from e
    except KeyError:
        raise KeyError("Missing 'data' key in JSON response")
```

**Changes Explained (one line each):**
- Replaced `requests` with `urllib.request` and `json` to satisfy the standard-library-only constraint.
- Added `timeout=10` to prevent indefinite hangs and improve network call correctness.
- Used a `with` statement to guarantee automatic connection cleanup after use.
- Placed `HTTPError` catch before `URLError` to correctly distinguish server responses from transport failures.
- Mapped network/timeout failures to `RuntimeError` for clear caller distinction.
- Mapped JSON parsing failures to `ValueError` to separate data format issues from network issues.
- Mapped missing key failures to `KeyError` with a descriptive message for immediate identification.
- Used `from e` exception chaining to preserve original tracebacks without swallowing errors.

**Explicit Failure Behaviors:**
- **URL times out:** Raises a `RuntimeError` with the message `"Network or timeout error: timed out"` (preserving the original traceback via chaining).
- **Response is not JSON:** Raises a `ValueError` with the message `"Response is not valid JSON: <decode error details>"` (preserving the original traceback).
- **JSON has no "data" key:** Raises a `KeyError` with the message `"Missing 'data' key in JSON response"`.
