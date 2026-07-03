# 27B_Heretic_NEO_CODE_IQ4_XS — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here’s a clean, production-ready refactor:

```python
import requests
from requests.exceptions import HTTPError, RequestException
from typing import Any


class DataFetchError(RuntimeError):
    """Raised when fetching or parsing remote data fails."""


def fetch_data(url: str, timeout: int = 10) -> Any:
    """
    Fetch JSON from `url` and return the value of the "data" field.

    Raises:
        DataFetchError: If the request fails, response is not valid JSON,
                        or the "data" key is missing.
    """
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()
    except RequestException as e:
        raise DataFetchError(f"Request failed for {url}: {e}") from e

    try:
        payload = response.json()
    except ValueError as e:
        raise DataFetchError(f"Invalid JSON from {url}: {e}") from e

    if "data" not in payload:
        raise DataFetchError(f"Response from {url} missing 'data' field.")

    return payload["data"]
```

Key improvements:
- Clear naming: `fetch_data(url)` instead of `d(u)`.
- Proper error handling:
  - Network/timeout errors → `RequestException`
  - HTTP errors (4xx/5xx) → `raise_for_status()`
  - Bad JSON → `ValueError`
  - Missing `"data"` key → explicit check
- Uses a custom exception (`DataFetchError`) so callers can catch fetch-specific issues.
- Adds a reasonable timeout to avoid hanging.
- Type hints and docstring for readability and tooling support.

If you prefer a simpler version without a custom exception, I can provide that too.
