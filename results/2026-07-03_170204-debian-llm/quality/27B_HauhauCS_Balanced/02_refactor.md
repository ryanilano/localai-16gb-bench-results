# 27B_HauhauCS_Balanced — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactor with clear naming, proper error handling, timeouts, type hints, and documentation:

```python
import requests
from typing import Any

def fetch_data(url: str, timeout: int = 10) -> Any:
    """
    Fetches and returns the 'data' field from a JSON API response.

    Args:
        url: The API endpoint to request.
        timeout: Timeout in seconds for the request (default: 10).

    Returns:
        The value associated with the 'data' key in the JSON response.

    Raises:
        RuntimeError: If the HTTP request fails (network, timeout, or non-2xx status).
        ValueError: If the response body is not valid JSON.
        KeyError: If the 'data' key is missing from the JSON response.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx responses
        return response.json()["data"]
    except requests.exceptions.RequestException as e:
        raise RuntimeError(f"Request failed: {e}") from e
    except ValueError as e:
        raise ValueError(f"Invalid JSON response: {e}") from e
    except KeyError as e:
        raise KeyError(f"Missing 'data' key in response: {e}") from e
```

### 🔑 Key Improvements:
1. **Descriptive Naming**: `d(u)` → `fetch_data(url)` for immediate readability.
2. **Module-Level Import**: Moved `import requests` outside the function to avoid repeated overhead and follow PEP 8.
3. **Timeout Parameter**: Added `timeout=10` to prevent hanging indefinitely on slow/unresponsive endpoints.
4. **HTTP Status Checking**: `response.raise_for_status()` explicitly catches 4xx/5xx errors instead of silently treating them as success.
5. **Granular Error Handling**:
   - `RequestException`: Catches network, connection, and HTTP errors.
   - `ValueError`: Catches malformed JSON responses.
   - `KeyError`: Catches missing `"data"` key.
   - All exceptions use `from e` for proper exception chaining (preserves tracebacks).
6. **Type Hints & Docstring**: Clarifies expected inputs, outputs, and possible exceptions for better IDE support and maintainability.

### 💡 Usage Example:
```python
try:
    result = fetch_data("https://api.example.com/endpoint")
    print(result)
except RuntimeError as e:
    print(f"Network/HTTP issue: {e}")
except ValueError as e:
    print(f"Bad JSON: {e}")
except KeyError as e:
    print(f"Unexpected API structure: {e}")
```

### 📝 Note:
If you prefer the function to return `None` or a default value on failure instead of raising exceptions, you can adjust the `except` blocks to `return None` or log the error. However, raising explicit exceptions is generally preferred for I/O operations as it forces the caller to handle failures explicitly.
