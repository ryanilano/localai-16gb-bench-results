# 27B_Heretic_NEO_CODE_IQ3_M — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here’s a clean, production-ready refactor:

```python
import requests
from requests.exceptions import HTTPError, RequestException

def fetch_data(url: str, timeout: float = 10.0) -> dict:
    """
    Fetch JSON from `url` and return the "data" field.

    Raises:
        RuntimeError: For network or HTTP-related failures.
        ValueError:   If the response is not JSON or "data" is missing/invalid.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()
    except HTTPError as e:
        raise RuntimeError(
            f"HTTP error {e.response.status_code} while fetching {url}"
        ) from e
    except RequestException as e:
        raise RuntimeError(
            f"Network error while fetching {url}: {e}"
        ) from e

    try:
        payload = response.json()
    except ValueError as e:
        raise RuntimeError(
            f"Invalid JSON from {url}: {e}"
        ) from e

    if "data" not in payload or not isinstance(payload["data"], dict):
        raise ValueError(
            f"Missing or invalid 'data' field in response from {url}"
        )

    return payload["data"]
```

Key improvements:
- Meaningful names: `fetch_data`, `url`, `timeout`, `payload`.
- Import moved to top level; no inline imports.
- Type hints and docstring clarify intent and behavior.
- Proper error handling:
  - `HTTPError`: non-2xx responses.
  - `RequestException`: timeouts, connection failures, etc.
  - `ValueError` (JSON decode) and missing/invalid `"data"` key.
- Uses `raise_for_status()` and exception chaining (`from e`) for better debugging.
- Adds a `timeout` parameter (best practice to avoid hanging).

If you prefer not to raise (e.g., for background tasks), I can show a variant that logs and returns `None` or a structured error instead.
