# 27B_Heretic_NEO_CODE_IQ4_XS — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here’s a cleaner, production-ready version with proper error handling, timeouts, and readable naming:

```python
import requests
from typing import Any, Dict

def fetch_api_data(url: str, timeout: int = 5) -> Dict[str, Any]:
    """
    Fetch and return the 'data' field from a JSON API response.

    Raises:
        RuntimeError: If the request fails, returns invalid JSON, or lacks the 'data' field.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()
    except requests.RequestException as exc:
        raise RuntimeError(f"Request failed for {url}: {exc}") from exc

    try:
        payload = response.json()
    except ValueError as exc:
        raise RuntimeError(f"Invalid JSON from {url}: {exc}") from exc

    if "data" not in payload:
        raise RuntimeError(f"Response from {url} is missing the 'data' field")

    return payload["data"]
```

Key improvements:
- Meaningful names: `fetch_api_data(url)` instead of `d(u)`.
- Import at module level: avoids repeated imports and follows Python conventions.
- Timeout: prevents hanging on slow/unresponsive endpoints.
- HTTP error handling: `raise_for_status()` catches 4xx/5xx responses.
- JSON error handling: safely handles malformed responses.
- Missing key handling: explicit check for `"data"` instead of silent KeyError.
- Clear, chainable error messages: makes debugging easier while preserving original exceptions.

If you prefer a more compact version (still safe), here it is:

```python
import requests

def fetch_api_data(url: str, timeout: int = 5):
    try:
        resp = requests.get(url, timeout=timeout)
        resp.raise_for_status()
        data = resp.json()
        if "data" not in data:
            raise KeyError("'data' field missing in response")
        return data["data"]
    except (requests.RequestException, ValueError, KeyError) as e:
        raise RuntimeError(f"Failed to fetch data from {url}: {e}") from e
```

Let me know if you’d like logging, custom exceptions, or async/aiohttp support added.
