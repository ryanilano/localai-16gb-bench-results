# 35B_UD-Q3_K_M — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactored version with clear structure, type hints, docstrings, and robust error handling:

```python
import requests
from requests.exceptions import RequestException

def fetch_data(url: str) -> dict:
    """
    Fetch and return the 'data' field from a JSON API response.
    
    Args:
        url: The URL to fetch.
        
    Returns:
        The 'data' dictionary from the response.
        
    Raises:
        RequestException: If the HTTP request fails.
        ValueError: If the response contains invalid JSON.
        KeyError: If the 'data' key is missing from the JSON response.
    """
    try:
        response = requests.get(url)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx status codes
        data = response.json()       # Raises ValueError on invalid JSON
    except ValueError as e:
        raise ValueError(f"Invalid JSON response from {url}") from e
    except RequestException as e:
        raise RequestException(f"HTTP request failed for {url}: {e}") from e

    try:
        return data["data"]
    except KeyError:
        available_keys = list(data.keys()) if isinstance(data, dict) else "N/A"
        raise KeyError(f"Missing 'data' key in response. Available keys: {available_keys}") from None
```

### 🔑 Key Improvements:
1. **Module-level imports**: `import requests` is moved to the top (PEP 8 standard).
2. **Status code validation**: `response.raise_for_status()` catches `4xx`/`5xx` errors before attempting JSON parsing.
3. **Explicit error handling**:
   - `ValueError` covers invalid JSON responses
   - `RequestException` covers network timeouts, DNS failures, connection errors, etc.
   - `KeyError` handles missing keys with a helpful message showing available keys
4. **Exception chaining**: `from e` and `from None` preserve tracebacks for easier debugging.
5. **Type hints & docstring**: Improves IDE support, readability, and API contract clarity.
6. **Descriptive naming**: `fetch_data` clearly indicates the function's purpose.

### 💡 Usage Example:
```python
try:
    result = fetch_data("https://api.example.com/data")
    print(result)
except (RequestException, ValueError, KeyError) as e:
    print(f"Failed to fetch data: {e}")
```

### 🔄 Alternative (Silent Failure):
If you prefer returning `None` instead of raising exceptions (common in internal utilities), you can replace the `try/except` blocks with:
```python
def fetch_data_safe(url: str) -> dict | None:
    try:
        response = requests.get(url)
        response.raise_for_status()
        data = response.json()
        return data.get("data")
    except (RequestException, ValueError):
        return None
```

Choose the raising version for APIs/libraries where failures should be explicit, and the `None` version for internal helpers where graceful degradation is preferred. Let me know if you need timeout settings, retries, or async support added!
