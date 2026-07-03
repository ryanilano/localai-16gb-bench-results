# 27B_HauhauCS_Balanced — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, runnable solution with the function and 3 unit tests using Python's built-in `unittest` module:

```python
def is_palindrome(s: str) -> bool:
    """Check if a string is a palindrome, ignoring case and non-alphanumeric characters."""
    cleaned = [c.lower() for c in s if c.isalnum()]
    return cleaned == cleaned[::-1]


import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_valid_palindrome_with_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        self.assertFalse(is_palindrome("race a car"))

    def test_edge_cases_empty_and_symbols_only(self):
        self.assertTrue(is_palindrome(""))
        self.assertTrue(is_palindrome("...,,!!"))

if __name__ == "__main__":
    unittest.main()
```

### How it works:
1. **Cleaning**: `[c.lower() for c in s if c.isalnum()]` filters out spaces, punctuation, and symbols while converting remaining characters to lowercase.
2. **Palindrome check**: `cleaned == cleaned[::-1]` compares the cleaned list with its reverse. Lists compare element-by-element in Python, so this efficiently checks for palindrome symmetry.
3. **Unit tests**: 
   - Test 1: Validates handling of mixed case and punctuation
   - Test 2: Confirms non-palindromes return `False`
   - Test 3: Covers edge cases (empty string and strings with only non-alphanumeric characters, which are trivially palindromes)

Run the file directly (`python script.py`) to execute the tests. All three should pass.
