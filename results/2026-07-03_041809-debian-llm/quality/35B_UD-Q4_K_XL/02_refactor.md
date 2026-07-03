# 35B_UD-Q4_K_XL — 02_refactor

## Prompt

Refactor this for clarity and proper error handling:
def d(u):
 import requests;return requests.get(u).json()["data"]

## Response

Here's a refactored version that follows Python best practices, improves readability, and adds robust error handling:

```python
import requests
from typing import Any

def fetch_data(url: str, timeout: int = 10) -> Any:
    """
    Fetch JSON data from a URL and return the value of the 'data'
