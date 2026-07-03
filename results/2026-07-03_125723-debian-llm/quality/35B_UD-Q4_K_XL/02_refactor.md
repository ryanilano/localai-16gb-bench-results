# 35B_UD-Q4_K_XL — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a refactored version that follows Python best practices, improves readability, and adds robust error handling:

```python
import requests
from typing import Any

def fetch_data(url: str, timeout: int = 10) -> Any:
    """
    Fetch JSON data from a URL and return the value associated with the 'data' key.
    
    Args:
        url: The URL to fetch data from.
        timeout: Request timeout in seconds (default: 10).
        
    Returns:
        The value of the 'data' key from the JSON response.
        
    Raises:
        requests.exceptions.HTTPError: If the HTTP request returned an unsuccessful status code.
        requests.exceptions.RequestException: For network-related errors.
        ValueError: If the response body is not valid JSON.
        KeyError: If the 'data' key is missing from the response JSON.
    """
    response = requests.get(url, timeout=timeout)
    response.raise_for_status()
    
    try:
        json_data = response.json()
    except ValueError as e:
        raise ValueError(f"Invalid JSON response from {url}: {e}") from e
        
    if "data" not in json_data:
        raise KeyError(
            f"Missing 'data' key in JSON response from {url}. "
            f"Available keys: {list(json_data.keys())}"
        )
        
    return json_data["data"]
```

### Key Improvements:
1. **Moved imports to the top**: Imports inside functions are inefficient and against PEP 8.
2. **Descriptive naming**: `d(u)` → `fetch_data(url)` for clarity.
3. **Type hints & docstring**: Makes the function self-documenting and IDE-friendly.
4. **Timeout parameter**: Prevents the script from hanging indefinitely on unresponsive servers.
5. **Explicit error handling**:
   - `raise_for_status()` converts HTTP errors (4xx/5xx) into exceptions.
   - `try/except ValueError` catches malformed JSON responses.
   - Explicit `KeyError` with available keys helps debug API structure changes.
   - `from e` preserves the original traceback for easier debugging.
6. **PEP 8 compliance**: Proper line breaks, spacing, and structure.

### Usage Example:
```python
try:
    result = fetch_data("https://api.example.com/data")
    print(result)
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
except ValueError as e:
    print(f"JSON parsing error: {e}")
except KeyError as e:
    print(f"API structure changed: {e}")
except requests.exceptions.RequestException as e:
    print(f"Network error: {e}")
```

### Alternative (Graceful Fallback):
If you prefer returning `None` instead of raising exceptions, replace the end of the function with:
```python
    try:
        return response.json().get("data")
    except ValueError:
        return None
```

Choose the exception-raising version for libraries/APIs where failures should be explicit, or the fallback version for scripts where missing data should be handled silently.
