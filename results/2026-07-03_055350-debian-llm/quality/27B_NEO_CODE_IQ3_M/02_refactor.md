# 27B_NEO_CODE_IQ3_M — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactor with clear naming, type hints, documentation, and robust error handling:

```python
import requests
from typing import Any

def fetch_api_data(url: str, timeout: int = 10) -> Any:
    """Fetch JSON response from a URL and return the 'data' field.

    Args:
        url: The API endpoint URL.
        timeout: Request timeout in seconds.

    Returns:
        The value associated with the 'data' key in the JSON response.

    Raises:
        RuntimeError: For network failures or HTTP errors (4xx/5xx).
        ValueError: If the response is not valid JSON or missing 'data'.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx status codes
        response_data = response.json()
        return response_data["data"]
    except requests.exceptions.RequestException as e:
        raise RuntimeError(f"Failed to fetch data from {url}: {e}") from e
    except (ValueError, KeyError) as e:
        raise ValueError(f"Invalid response format from {url}: {e}") from e
```

### 🔑 Key Improvements:
| Issue in Original | Fix Applied |
|------------------|-------------|
| `d(u)` cryptic naming | Descriptive `fetch_api_data(url)` |
| Inline import & one-liner | Top-level import, proper structure, PEP 8 formatting |
| No error handling | Catches network/HTTP errors, JSON decode failures, and missing keys |
| Silent failures on bad status codes | `response.raise_for_status()` explicitly fails on 4xx/5xx |
| No timeout | Added `timeout` parameter (prevents hanging connections) |
| No documentation | Added docstring, type hints, and clear raise contracts |

### 💡 Usage Example:
```python
try:
    result = fetch_api_data("https://api.example.com/resource")
    print(result)
except RuntimeError as e:
    print(f"Network/Server issue: {e}")
except ValueError as e:
    print(f"Payload issue: {e}")
```

### 🛠 Optional Enhancements:
- **Custom Exceptions**: Replace `RuntimeError`/`ValueError` with project-specific exceptions if you have an exception hierarchy.
- **Session Reuse**: If calling multiple endpoints, wrap in `requests.Session()` for connection pooling:
  ```python
  with requests.Session() as session:
      data = fetch_api_data(url, session=session)  # would require minor signature tweak
  ```
- **Return Type Precision**: Change `-> Any` to `-> dict` or `-> list` if the API contract guarantees a specific structure.

Let me know if you'd like this adapted for async (`aiohttp`), typed responses (`pydantic`), or integrated into a larger service layer!
