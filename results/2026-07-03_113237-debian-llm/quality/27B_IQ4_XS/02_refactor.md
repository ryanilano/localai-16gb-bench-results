# 27B_IQ4_XS — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactoring with clear naming, proper structure, type hints, and robust error handling:

```python
import requests
from typing import Any

def fetch_data(url: str, timeout: int = 10) -> Any:
    """
    Fetch JSON data from the given URL and return the 'data' field.

    Args:
        url: The API endpoint to request.
        timeout: Request timeout in seconds.

    Returns:
        The value associated with the 'data' key in the JSON response.

    Raises:
        RuntimeError: If the HTTP request fails (network, timeout, or HTTP error).
        ValueError: If the response is not valid JSON or missing the 'data' key.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx status codes
        json_data = response.json()
        return json_data["data"]
    except requests.exceptions.RequestException as e:
        raise RuntimeError(f"Failed to fetch data from {url}: {e}") from e
    except (KeyError, ValueError, requests.exceptions.JSONDecodeError) as e:
        raise ValueError(f"Invalid response from {url}: {e}") from e
```

### 🔑 Key Improvements:
| Original Issue | Fix |
|----------------|-----|
| `d`, `u` naming | Renamed to `fetch_data`, `url` for readability |
| One-liner with `;` | Split into logical steps following PEP 8 |
| No error handling | Added targeted `try/except` blocks with exception chaining (`from e`) |
| Silent HTTP errors | `response.raise_for_status()` catches 4xx/5xx responses |
| No timeout | Added `timeout=10` to prevent hanging requests |
| Import inside function | Moved to module level (best practice) |
| No documentation | Added docstring, type hints, and clear raise conditions |

### 💡 Usage Notes:
- **Exception Chaining**: `from e` preserves the original traceback, making debugging easier.
- **Error Granularity**: You can split `RequestException` into `HTTPError`, `ConnectionError`, or `Timeout` if your app needs different handling per error type.
- **Production Tip**: Replace `raise` with `logging.error()` if you prefer graceful degradation over crashing:
  ```python
  import logging
  logger = logging.getLogger(__name__)
  # ... inside except blocks:
  logger.error("Failed to fetch %s: %s", url, e)
  return None  # or raise a custom exception
  ```

Let me know if you need async support, retry logic, or integration with a specific framework!
