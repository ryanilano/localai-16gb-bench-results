# 27B_IQ4_XS — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a refactored version that improves readability, adds type hints, documents behavior, and handles common failure modes gracefully:

```python
import requests
from typing import Any

def fetch_api_data(url: str, timeout: int = 10) -> Any:
    """
    Fetches the 'data' field from a JSON API response.

    Args:
        url: The endpoint URL to request.
        timeout: Maximum time in seconds to wait for the response.

    Returns:
        The value associated with the 'data' key in the JSON response.

    Raises:
        RuntimeError: If the HTTP request fails or returns an error status.
        ValueError: If the response body is not valid JSON.
        KeyError: If the JSON response does not contain a 'data' key.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx status codes
        json_data = response.json()
        return json_data["data"]
    except requests.exceptions.HTTPError as e:
        raise RuntimeError(f"HTTP error {e.response.status_code}: {e}") from e
    except requests.exceptions.RequestException as e:
        raise RuntimeError(f"Request failed: {e}") from e
    except ValueError as e:
        raise ValueError(f"Invalid JSON in response: {e}") from e
    except KeyError:
        raise KeyError("Expected 'data' key not found in JSON response.")
```

### 🔑 Key Improvements:
1. **Descriptive Naming**: `d` → `fetch_api_data`, `u` → `url`
2. **Proper Structure**: Multi-line function with clear separation of concerns
3. **Type Hints & Docstring**: Makes the function self-documenting and IDE-friendly
4. **Timeout**: Prevents indefinite hanging on slow/unresponsive endpoints
5. **Comprehensive Error Handling**:
   - `HTTPError`: Catches 4xx/5xx responses via `raise_for_status()`
   - `RequestException`: Catches network failures, timeouts, DNS errors, etc.
   - `ValueError`: Catches malformed JSON responses
   - `KeyError`: Catches missing `"data"` key
   - Uses `from e` to preserve the original traceback for debugging
6. **Import Placement**: Moved to module level (best practice)

### 💡 Production Notes:
- In larger applications, consider using `logging` instead of raising exceptions, or create custom domain exceptions (e.g., `APIDataFetchError`).
- If calling this function repeatedly, reuse a `requests.Session()` for connection pooling and better performance.
- If you know the expected type of `"data"`, replace `Any` with a specific type (e.g., `dict[str, Any]` or a Pydantic model).

Let me know if you'd like a version with logging, session reuse, or async support!
