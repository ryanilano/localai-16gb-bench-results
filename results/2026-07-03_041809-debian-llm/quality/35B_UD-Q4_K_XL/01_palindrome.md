# 35B_UD-Q4_K_XL — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, runnable implementation with the function and 3 unit tests using Python's built-in `unittest` module:

```python
def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Check if the cleaned string reads the same forwards and backwards
    return cleaned == cleaned[::-1]


import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_simple_palindrome(self):
        self.assertTrue(is_palindrome("racecar"))

    def test_palindrome_with_mixed_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        self.assertFalse(is_palindrome("hello"))

if __name__ == '__main__':
    unittest.main()
```

### Key Features:
- **Efficiency**: Uses a generator expression inside `''.join()` for O(n) time and O(n) space complexity.
- **Robustness**: `str.isalnum()` correctly filters out spaces, punctuation, and symbols while keeping letters and digits.
- **Case Insensitivity**: `.lower()` ensures uppercase and lowercase letters are treated identically.
- **Unit Tests**: Covers a standard palindrome, a classic real-world example with punctuation/casing, and a clear non-palindrome case.

Run it directly with `python script_name.py` to see the tests pass.
