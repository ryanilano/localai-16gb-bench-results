# 27B_HauhauCS_Balanced — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, ready-to-run Python script with the function and 3 unit tests using the built-in `unittest` module:

```python
import unittest

def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter to keep only alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Compare the cleaned string with its reverse
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    
    def test_valid_palindrome_with_mixed_case_and_punctuation(self):
        """Test a classic palindrome with spaces, punctuation, and mixed case."""
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))
        
    def test_non_palindrome(self):
        """Test a string that is not a palindrome."""
        self.assertFalse(is_palindrome("race a car"))
        
    def test_empty_and_non_alphanumeric_only(self):
        """Test edge cases: empty string and string with only special characters."""
        self.assertTrue(is_palindrome(""))
        self.assertTrue(is_palindrome("   ,,!!  "))


if __name__ == "__main__":
    unittest.main()
```

### How it works:
1. **Filtering & Normalization**: `''.join(char.lower() for char in s if char.isalnum())` keeps only letters and digits, converts them to lowercase, and joins them back into a string.
2. **Palindrome Check**: `cleaned == cleaned[::-1]` compares the cleaned string to its reverse in `O(n)` time.
3. **Unit Tests**: Uses Python's standard `unittest` framework. Run the script directly to execute all 3 tests automatically.

### Expected Output when run:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```
