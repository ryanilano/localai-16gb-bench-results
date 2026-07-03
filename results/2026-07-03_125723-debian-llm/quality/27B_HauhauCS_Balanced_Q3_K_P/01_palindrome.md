# 27B_HauhauCS_Balanced_Q3_K_P — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, ready-to-run Python script with the function and 3 unit tests:

```python
import unittest

def is_palindrome(s: str) -> bool:
    """
    Checks if a string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    
    def test_valid_palindrome_with_mixed_case_and_punctuation(self):
        """Test a classic palindrome with mixed case and punctuation."""
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))
        
    def test_invalid_palindrome(self):
        """Test a string that is not a palindrome."""
        self.assertFalse(is_palindrome("race a car"))
        
    def test_empty_and_whitespace_only(self):
        """Test edge cases: empty string and whitespace-only strings."""
        self.assertTrue(is_palindrome(""))
        self.assertTrue(is_palindrome("   \t\n  "))


if __name__ == "__main__":
    unittest.main()
```

### How it works:
1. **Cleaning**: `''.join(char.lower() for char in s if char.isalnum())` filters out spaces, punctuation, and symbols while converting everything to lowercase.
2. **Comparison**: `cleaned == cleaned[::-1]` checks if the cleaned string reads the same forwards and backwards using Python's slice reversal.
3. **Tests**: The `unittest` class covers three scenarios:
   - A valid palindrome with mixed casing and punctuation
   - A non-palindrome
   - Edge cases (empty/whitespace-only strings, which are technically palindromes)

Run the script directly (`python script.py`) to execute the tests. All 3 should pass.
