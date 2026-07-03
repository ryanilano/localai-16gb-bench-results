# 35B_UD-Q3_K_M — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a production-ready refactoring that improves clarity, safety, and maintainability:

```python
import requests
from typing import Any

def fetch_data(url: str) -> Any:
    """
    Fetch and return the 'data' field from a JSON API response.
    
    Args:
        url: The URL to request.
        
    Returns:
        The value associated with the 'data' key.
        
    Raises:
        requests.RequestException: On HTTP or network errors.
        ValueError: If the response is not valid JSON.
        KeyError: If the 'data' key is missing from the response.
    """
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    
    try:
        payload = response.json()
    except requests.exceptions.JSONDecodeError as e:
        raise ValueError(f"Failed to parse JSON response: {e}") from e
        
    return payload["data"]
```

### Key Improvements:
1. **Top-level imports**: `import requests` is moved to the top (importing inside functions is an anti-pattern).
2. **Explicit error handling**:
   - `response.raise_for_status()` catches HTTP errors (4xx, 5xx) instead of silently returning bad data.
   - `timeout=10` prevents the script from hanging indefinitely on unresponsive servers.
   - JSON parsing is wrapped in `try/except` to provide clear error messages instead of cryptic tracebacks.
3. **Type hints & docstring**: Makes the function's purpose, inputs, outputs, and failure modes explicit for IDE support and documentation.
4. **Readable function name**: Changed from `d` to `fetch_data` to follow PEP 8 naming conventions.
5. **Preserves original behavior**: Still raises `KeyError` if `"data"` is missing, but now with predictable behavior and proper error chaining (`from e`).

### Usage Example:
```python
try:
    result = fetch_data("https://api.example.com/data")
    print(result)
except requests.RequestException as e:
    print(f"Network/HTTP error: {e}")
except ValueError as e:
    print(f"Invalid JSON: {e}")
except KeyError as e:
    print(f"Missing 'data' key: {e}")
```

> 💡 **Note**: Requires `requests` to be installed: `pip install requests`

Let me know if you'd prefer a version that returns a default value (e.g., `None`) instead of raising on missing keys!
