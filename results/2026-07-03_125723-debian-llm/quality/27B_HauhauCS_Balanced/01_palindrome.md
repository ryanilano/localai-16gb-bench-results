# 27B_HauhauCS_Balanced — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, Pythonic implementation with 3 unit tests using the built-in `unittest` module:

```python
import unittest

def is_palindrome(s: str) -> bool:
    """
    Checks if a string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter to only alphanumeric characters and convert to lowercase
    cleaned = [char.lower() for char in s if char.isalnum()]
    # Compare the cleaned sequence with its reverse
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_valid_palindrome_with_punctuation_and_case(self):
        """Test a classic palindrome with mixed case and punctuation."""
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        """Test a string that is not a palindrome."""
        self.assertFalse(is_palindrome("race a car"))

    def test_edge_cases_empty_and_symbols_only(self):
        """Test edge cases: empty string and string with only non-alphanumeric chars."""
        self.assertTrue(is_palindrome(""))
        self.assertTrue(is_palindrome(".,!?"))


if __name__ == "__main__":
    unittest.main()
```

### Key Details:
- **`char.isalnum()`**: Efficiently filters out spaces, punctuation, and special characters
- **`char.lower()`**: Ensures case-insensitivity
- **`cleaned == cleaned[::-1]`**: Pythonic way to check for palindrome property
- **Time Complexity**: O(n) where n is the length of the string
- **Space Complexity**: O(n) for the filtered list (can be reduced to O(1) with a two-pointer approach if memory is constrained)

### How to Run:
Save the code in a file (e.g., `palindrome_test.py`) and run:
```bash
python palindrome_test.py -v
```
The `-v` flag will show detailed test results. All 3 tests should pass.
