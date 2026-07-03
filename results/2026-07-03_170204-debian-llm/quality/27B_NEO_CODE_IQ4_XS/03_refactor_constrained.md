# 27B_NEO_CODE_IQ4_XS — 03_refactor_constrained

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
            data = json.loads(response.read())
        return data["data"]
    except urllib.error.URLError as e:
        raise RuntimeError(f"Network/timeout error fetching {u}: {e}") from e
    except json.JSONDecodeError as e:
        raise ValueError(f"Response is not valid JSON: {e}") from e
    except KeyError as e:
        raise KeyError(f"JSON payload missing 'data' key: {e}") from e
```

**Explanation of changes (one line each):**
- Replaced `requests` with `urllib.request` and `json` to comply with the standard-library-only constraint.
- Added a 10-second timeout to `urlopen` to prevent indefinite blocking on unresponsive servers.
- Used a context manager (`with`) to guarantee the HTTP connection is closed even if parsing fails.
- Added explicit exception handling that catches specific errors and re-raises them with descriptive messages, ensuring callers can distinguish failure modes.
- Preserved the original JSON parsing and dictionary lookup logic to maintain identical behavior for valid inputs.

**Explicit failure behavior:**
- **URL times out:** Raises a `RuntimeError` (chaining the underlying `URLError`) with a clear message indicating a network or timeout failure.
- **Response is not JSON:** Raises a `ValueError` (chaining `JSONDecodeError`) explicitly stating the response failed JSON parsing.
- **JSON has no "data" key:** Raises a `KeyError` (with chained context) explicitly indicating the parsed JSON payload lacks the required key.
