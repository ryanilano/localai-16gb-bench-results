# 27B_NEO_CODE_IQ3_M — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a refactored version with proper naming, type hints, timeouts, and comprehensive error handling:

```python
import requests

def fetch_data(url: str, timeout: int = 10) -> dict:
    """
    Fetch JSON from a URL and return the 'data' field.
    
    Args:
        url: The endpoint to request.
        timeout: Request timeout in seconds.
        
    Returns:
        The value of the 'data' key from the JSON response.
        
    Raises:
        RuntimeError: For network or HTTP errors.
        ValueError: If the response is not valid JSON.
        KeyError: If the 'data' key is missing from the response.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx responses
        json_payload = response.json()
        return json_payload["data"]
    except requests.exceptions.HTTPError as e:
        raise RuntimeError(f"HTTP error {response.status_code}: {e}") from e
    except requests.exceptions.ConnectionError as e:
        raise RuntimeError(f"Connection error: {e}") from e
    except requests.exceptions.Timeout as e:
        raise RuntimeError(f"Request timed out after {timeout}s: {e}") from e
    except requests.exceptions.RequestException as e:
        raise RuntimeError(f"Unexpected request error: {e}") from e
    except ValueError as e:
        raise ValueError("Response is not valid JSON") from e
    except KeyError:
        raise KeyError("Expected 'data' key not found in response") from KeyError("data")
```

### Key Improvements:
1. **Descriptive Naming**: `d(u)` → `fetch_data(url)` makes intent immediately clear.
2. **Top-Level Import**: Moved `import requests` outside the function (best practice; avoids repeated import overhead and improves readability).
3. **Timeout**: Added `timeout=10` to prevent hanging indefinitely on slow/unresponsive servers.
4. **HTTP Status Validation**: `response.raise_for_status()` converts 4xx/5xx responses into explicit `HTTPError` exceptions.
5. **Granular Error Handling**:
   - Network/connection/timeout errors are caught separately for clearer diagnostics.
   - `ValueError` catches malformed JSON (covers `requests.exceptions.JSONDecodeError`).
   - `KeyError` safely handles missing `"data"` fields.
   - `from e` preserves the original traceback for debugging.
6. **Type Hints & Docstring**: Improves IDE support, readability, and serves as inline documentation.

### Production Notes:
- If this is part of a larger service, consider replacing `raise RuntimeError(...)` with `logging.exception(...)` or raising custom exceptions (e.g., `APIDataFetchError`).
- For resilience, wrap calls with a retry library like `tenacity` or `urllib3.util.Retry`.
- If the API might return a non-dict JSON root (e.g., a list), add a type check: `if not isinstance(json_payload, dict): raise TypeError(...)` before accessing `["data"]`.
