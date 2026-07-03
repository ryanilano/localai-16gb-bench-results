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

Here is the refactored function.

```python
import requests

def fetch_json(u):
    try:
        # Added timeout to prevent indefinite blocking on slow connections
        response = requests.get(u, timeout=10)
        # Explicitly check for HTTP errors (4xx, 5xx)
        response.raise_for_status()
        # Parse JSON; raises ValueError if the content is not valid JSON
        payload = response.json()
        # Return the specific data field
        return payload["data"]
    except requests.exceptions.RequestException as e:
        # Distinguish network or HTTP failures from logic errors
        raise RuntimeError(f"Request failed for {u}: {e}") from e
    except ValueError as e:
        # Distinguish JSON parsing errors from other failures
        raise ValueError(f"Invalid JSON response: {e}") from e
    except KeyError as e:
        # Distinguish missing key errors from parsing errors
        raise KeyError(f"Missing 'data' key in response: {e}") from e
```

### Explanation of Changes
1.  **Moved import**: Relocated `import requests` to the top level to avoid re-importing on every function call.
2.  **Added timeout**: Included `timeout=10` in `requests.get` to ensure the function fails fast rather than hanging indefinitely.
3.  **Status check**: Added `raise_for_status()` to catch and handle HTTP error codes (like 404 or 500) explicitly.
4.  **Try/Except block**: Wrapped logic in a block to catch specific exceptions and re-raise them with context rather than letting them crash silently or obscurely.
5.  **RequestException handling**: Catches network issues (connection errors, timeouts) and HTTP errors, converting them to a `RuntimeError`.
6.  **ValueError handling**: Catches JSON decoding errors (since `requests.JSONDecodeError` inherits from `ValueError`), converting them to a clear `ValueError`.
7.  **KeyError handling**: Catches the specific case where the JSON is valid but lacks the required "data" key.

### Scenario Outcomes

*   **If the URL times out**: The function raises a `RuntimeError` (wrapping the `requests.exceptions.Timeout`).
*   **If the response is not JSON**: The function raises a `ValueError` (wrapping the `requests.exceptions.JSONDecodeError`).
*   **If the JSON has no "data" key**: The function raises a `KeyError` with a message indicating the missing key.
