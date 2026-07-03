# 35B_UD-Q4_K_XL — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, Pythonic implementation along with 3 unit tests using the built-in `unittest` framework:

```python
import unittest

def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter alphanumeric characters and convert to lowercase
    cleaned = [char.lower() for char in s if char.isalnum()]
    # Compare the list with its reverse
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_palindrome_with_punctuation_and_mixed_case(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome_with_spaces(self):
        self.assertFalse(is_palindrome("race a car"))

    def test_edge_cases(self):
        self.assertTrue(is_palindrome(""))
        self.assertTrue(is_palindrome("A"))
        self.assertTrue(is_palindrome("12321"))


if __name__ == "__main__":
    unittest.main()
```

### Key Features:
- **`char.isalnum()`**: Efficiently filters out spaces, punctuation, and special characters.
- **`.lower()`**: Normalizes case so comparisons are case-insensitive.
- **`cleaned == cleaned[::-1]`**: Pythonic way to check for palindrome symmetry.
- **Time Complexity**: `O(n)` where `n` is the length of the string (single pass to filter, single pass to compare).
- **Space Complexity**: `O(n)` for the filtered list (can be reduced to `O(1)` with a two-pointer approach if memory is strictly constrained).

Run the script directly with `python filename.py` to see the tests pass.
