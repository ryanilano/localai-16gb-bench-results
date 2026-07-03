# 27B_NEO_CODE_IQ4_XS — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's the complete implementation with the function and three unit tests using Python's built-in `unittest` module:

```python
def is_palindrome(s: str) -> bool:
    # Filter out non-alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Check if the cleaned string reads the same forwards and backwards
    return cleaned == cleaned[::-1]


import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_valid_palindrome_with_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))
        
    def test_non_palindrome(self):
        self.assertFalse(is_palindrome("hello world"))
        
    def test_empty_string(self):
        self.assertTrue(is_palindrome(""))

if __name__ == '__main__':
    unittest.main()
```

### How it works:
1. **Filtering & Normalization**: `''.join(char.lower() for char in s if char.isalnum())` iterates through the string, keeps only alphanumeric characters (`isalnum()`), converts them to lowercase, and joins them into a clean string.
2. **Palindrome Check**: `cleaned == cleaned[::-1]` compares the cleaned string with its reverse. If they match, it's a palindrome.
3. **Unit Tests**: 
   - Tests a classic palindrome with mixed case and punctuation.
   - Tests a standard non-palindrome.
   - Tests an edge case (empty string, which is trivially a palindrome).

Run the script directly to execute the tests. All three will pass.
