# 27B_NEO_CODE_IQ4_XS — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactoring with clear naming, type hints, defensive error handling, and best practices:

```python
import requests
from requests.exceptions import RequestException, JSONDecodeError

def fetch_api_data(url: str, timeout: int = 10) -> dict | list:
    """Fetch the 'data' payload from a JSON API endpoint.

    Args:
        url: The API endpoint URL.
        timeout: Request timeout in seconds.

    Returns:
        The value associated with the 'data' key in the JSON response.

    Raises:
        RuntimeError: On network/HTTP failures or invalid JSON.
        KeyError: If the 'data' key is missing or response structure is invalid.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx status codes
        return response.json()["data"]
    except RequestException as e:
        raise RuntimeError(f"Request failed for {url}: {e}") from e
    except JSONDecodeError as e:
        raise ValueError(f"Invalid JSON from {url}: {e}") from e
    except (KeyError, TypeError) as e:
        raise KeyError(f"Response from {url} does not contain a 'data' key: {e}") from e
```

### 🔑 Key Improvements
| Original Issue | Fix |
|----------------|-----|
| `d(u)` | Renamed to `fetch_api_data(url)` for self-documenting code |
| Import inside function | Moved to module level (standard practice) |
| No error handling | Catches network/HTTP errors, malformed JSON, and missing keys |
| Silent failures | Raises descriptive exceptions with `from e` to preserve tracebacks |
| No timeout | Added `timeout=10` to prevent hanging on slow/unresponsive servers |
| No documentation | Added docstring, type hints, and clear argument/return descriptions |

### 🛡️ Error Handling Strategy
- **`RequestException`**: Catches connection timeouts, DNS failures, and `raise_for_status()` HTTP errors (4xx/5xx)
- **`JSONDecodeError`**: Catches non-JSON responses (e.g., HTML error pages)
- **`KeyError` / `TypeError`**: Catches cases where the response is valid JSON but lacks `"data"` or isn't a dict/list
- All exceptions are re-raised with context using `from e` so debugging tools show the full chain

### 💡 Usage Example
```python
try:
    result = fetch_api_data("https://api.example.com/v1/resource")
    print(result)
except RuntimeError as e:
    print(f"Network/HTTP issue: {e}")
except ValueError as e:
    print(f"Malformed response: {e}")
except KeyError as e:
    print(f"Unexpected API structure: {e}")
```

> **Note**: `dict | list` syntax requires Python 3.10+. For older versions, use `from typing import Union` and change the return type to `-> Union[dict, list]`.

Let me know if you'd like this adapted for async (`aiohttp`), cached responses, or specific API schemas!
