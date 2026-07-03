# 27B_IQ4_XS — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactoring with descriptive naming, type hints, documentation, timeouts, and comprehensive error handling:

```python
import requests
from typing import Any

def fetch_api_data(url: str, timeout: float = 10.0) -> Any:
    """
    Fetch data from a JSON API and return the value of the 'data' key.

    Args:
        url: The API endpoint URL.
        timeout: Request timeout in seconds.

    Returns:
        The value associated with the 'data' key in the JSON response.

    Raises:
        requests.exceptions.HTTPError: If the server returns an unsuccessful status code.
        requests.exceptions.RequestException: For network-related issues (DNS, connection, timeout).
        ValueError: If the response is not valid JSON.
        KeyError: If the 'data' key is missing from the response.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()
        json_data = response.json()
        return json_data["data"]
    except requests.exceptions.HTTPError as e:
        raise ValueError(f"HTTP error {e.response.status_code} for {url}") from e
    except requests.exceptions.RequestException as e:
        raise ConnectionError(f"Network error fetching {url}: {e}") from e
    except ValueError as e:
        raise ValueError(f"Failed to parse JSON from {url}: {e}") from e
    except KeyError:
        raise KeyError("Response JSON does not contain the 'data' key.")
```

### 🔑 Key Improvements
| Original Issue | Fix |
|----------------|-----|
| `def d(u):` | Renamed to `fetch_api_data(url: str)` for clarity & type safety |
| `import requests` inside function | Moved to module level (imports should be top-level) |
| No timeout | Added `timeout=10.0` to prevent hanging on slow/unresponsive endpoints |
| Silent crashes | Added explicit `try/except` blocks with chained exceptions (`from e`) to preserve tracebacks |
| `response.json()["data"]` chaining | Split into two steps to isolate JSON parsing errors from missing key errors |
| No documentation | Added Google-style docstring with args, returns, and raises |

### 💡 Usage Example
```python
try:
    result = fetch_api_data("https://api.example.com/endpoint")
    print(result)
except ValueError as e:
    print(f"Client/JSON error: {e}")
except ConnectionError as e:
    print(f"Network error: {e}")
except KeyError as e:
    print(f"Missing data: {e}")
```

### 🛠 Production Tips
1. **Use `requests.Session()`** if making multiple calls to the same host (reuses TCP connections).
2. **Add retries** for transient failures using `urllib3.util.Retry` + `requests.adapters.HTTPAdapter`.
3. If the `"data"` key is optional, replace `json_data["data"]` with `json_data.get("data")` and handle `None` appropriately instead of raising `KeyError`.

Let me know if you'd like a version with automatic retries or session management!
