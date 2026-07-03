# 27B_IQ4_XS — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, Pythonic implementation along with 3 unit tests:

```python
def is_palindrome(s: str) -> bool:
    """
    Checks if a string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Compare the cleaned string with its reverse
    return cleaned == cleaned[::-1]


# ==========================
# Unit Tests
# ==========================
def run_tests():
    # Test 1: Standard palindrome with mixed case and punctuation
    assert is_palindrome("A man, a plan, a canal: Panama") == True, "Test 1 failed"
    
    # Test 2: Non-palindrome
    assert is_palindrome("race a car") == False, "Test 2 failed"
    
    # Test 3: String with only non-alphanumeric characters (empty after cleaning)
    assert is_palindrome("   ,.!?") == True, "Test 3 failed"
    
    print("✅ All 3 unit tests passed.")

if __name__ == "__main__":
    run_tests()
```

### Key Details:
- **`char.isalnum()`**: Efficiently filters out spaces, punctuation, and symbols.
- **`char.lower()`**: Ensures case-insensitivity.
- **`cleaned[::-1]`**: Python's slice notation for reversing a string in O(n) time.
- **Edge Case Handling**: Empty strings or strings with only punctuation/symbols correctly evaluate to `True` (an empty sequence reads the same forwards and backwards).

You can run this script directly, or drop the `assert` statements into `pytest`/`unittest` for automated test suites.
