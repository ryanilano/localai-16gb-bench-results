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
            payload = json.loads(response.read().decode('utf-8'))
        return payload["data"]
    except socket.timeout:
        raise TimeoutError(f"Request to {u} timed out")
    except urllib.error.URLError as e:
        raise ConnectionError(f"Failed to fetch {u}: {e.reason}")
    except json.JSONDecodeError as e:
        raise ValueError(f"Response from {u} is not valid JSON: {e}")
    except KeyError:
        raise KeyError(f"JSON response from {u} lacks the 'data' key")
```

**Explanation of changes (one line each):**
1. Replaced `requests` with `urllib.request` and `json` to strictly comply with the standard-library-only constraint.
2. Added `timeout=10` to `urlopen` to prevent indefinite blocking on unresponsive or slow endpoints.
3. Wrapped the network and parsing logic in a `try/except` block to explicitly route distinct failure modes to specific exception types.
4. Raised standard exceptions (`TimeoutError`, `ConnectionError`, `ValueError`, `KeyError`) with contextual messages so callers can distinguish failures.
5. Used a context manager (`with`) for `urlopen` to guarantee the underlying socket is properly closed after reading.
6. Moved imports to module level to follow PEP 8 conventions and eliminate per-call import overhead.

**Explicit failure behavior:**
- **URL times out:** Raises `TimeoutError` with a message indicating the target URL and that the request exceeded the 10-second limit.
- **Response is not JSON:** Raises `ValueError` (wrapping `json.JSONDecodeError`) with a message indicating the response body failed JSON parsing.
- **JSON has no "data" key:** Raises `KeyError` with a message explicitly stating that the parsed JSON object is missing the required `"data"` key.
