# 27B_NEO_CODE_IQ4_XS — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactored version with clear naming, type hints, documentation, and robust error handling:

```python
import requests
from typing import Any

def fetch_api_data(url: str, timeout: int = 10) -> Any:
    """
    Fetch JSON data from a URL and return the 'data' field.
    
    Args:
        url: The API endpoint to query.
        timeout: Maximum time to wait for a response (seconds).
        
    Returns:
        The value associated with the 'data' key in the JSON response.
        
    Raises:
        RuntimeError: If the HTTP request fails, times out, or returns a 4xx/5xx status.
        ValueError: If the response body is not valid JSON.
        KeyError: If the 'data' key is missing from the parsed JSON.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx responses
        return response.json()["data"]
    except requests.exceptions.RequestException as e:
        raise RuntimeError(f"Request failed for {url}: {e}") from e
    except ValueError as e:
        raise ValueError(f"Invalid JSON response from {url}: {e}") from e
    except KeyError:
        raise KeyError(f"'data' key missing in response from {url}")
```

### 🔑 Key Improvements
| Original Issue | Fix Applied |
|----------------|-------------|
| `import` inside function | Moved to module level (follows PEP 8, avoids repeated overhead) |
| Opaque names (`d`, `u`) | Descriptive names (`fetch_api_data`, `url`) |
| No timeout | Added `timeout=10` to prevent hanging indefinitely |
| Silent HTTP errors | `response.raise_for_status()` catches 4xx/5xx status codes |
| Unhandled exceptions | Explicit `try/except` blocks for network, HTTP, JSON, and key errors |
| Lost exception context | `from e` chains exceptions for better debugging/tracing |
| No type hints/docstring | Added `typing.Any`, parameter types, and Google-style docstring |

### 💡 Usage Example
```python
try:
    result = fetch_api_data("https://api.example.com/v1/resource")
    print(result)
except RuntimeError as e:
    print(f"Network/HTTP error: {e}")
except ValueError as e:
    print(f"JSON parsing error: {e}")
except KeyError as e:
    print(f"API response mismatch: {e}")
```

### 🛠 Production Considerations
- **Multiple requests?** Use `requests.Session()` to reuse connections and set default headers/timeouts.
- **Flaky APIs?** Wrap calls with `tenacity` or `backoff` for automatic retries with exponential backoff.
- **Async needed?** Consider `httpx` or `aiohttp` if this runs in an async event loop.

Let me know if you'd like a version with retries, session management, or async support!
