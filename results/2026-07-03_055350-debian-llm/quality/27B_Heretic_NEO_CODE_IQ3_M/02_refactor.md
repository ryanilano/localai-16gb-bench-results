# 27B_Heretic_NEO_CODE_IQ3_M — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here’s a clean, production-ready refactor with proper error handling and readability:

```python
import requests
from typing import Any

class APIError(Exception):
    """Raised when an API request or response is invalid."""
    pass

def fetch_data(url: str, timeout: int = 10) -> Any:
    """
    Fetch JSON from `url` and return the value under the "data" key.

    Raises:
        APIError: If the request fails, returns a non-2xx status,
                  contains invalid JSON, or is missing the "data" key.
    """
    # 1. Make the request with a timeout
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # Raises HTTPError for 4xx/5xx
    except requests.RequestException as e:
        raise APIError(f"Request failed for {url}: {e}") from e

    # 2. Parse JSON
    try:
        json_data = response.json()
    except requests.JSONDecodeError as e:
        raise APIError(f"Invalid JSON from {url}: {e}") from e

    # 3. Extract the expected key
    if "data" not in json_data:
        raise APIError(f"Response from {url} is missing 'data' key")

    return json_data["data"]
```

Key improvements:
- Clear names: `fetch_data(url)` instead of `d(u)`.
- Import at module level, not inside the function.
- Added a timeout to avoid hanging.
- Handles:
  - Network/transport errors
  - HTTP error codes (4xx, 5xx)
  - Invalid JSON
  - Missing `"data"` field
- Uses a focused custom exception (`APIError`) so callers can distinguish API issues from unexpected bugs.
- Includes type hints and a docstring for maintainability.

If you prefer a simpler version without a custom exception, I can show a minimal variant that logs/prints instead.
