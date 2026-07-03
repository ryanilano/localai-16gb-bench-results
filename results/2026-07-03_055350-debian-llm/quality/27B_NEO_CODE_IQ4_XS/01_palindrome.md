# 27B_NEO_CODE_IQ4_XS — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's the complete solution with the function and three unit tests using Python's built-in `unittest` module:

```python
def is_palindrome(s):
    # Keep only alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Check if the cleaned string reads the same forwards and backwards
    return cleaned == cleaned[::-1]

import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        # Standard palindrome with spaces, punctuation, and mixed case
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        # Regular string that is not a palindrome
        self.assertFalse(is_palindrome("hello"))

    def test_empty_and_special_characters(self):
        # Edge cases: empty string and string with only non-alphanumeric chars
        self.assertTrue(is_palindrome(""))
        self.assertTrue(is_palindrome(".,;?!"))

if __name__ == '__main__':
    unittest.main()
```

### How it works:
1. **Filtering & Normalization**: `''.join(char.lower() for char in s if char.isalnum())` iterates through the input, keeps only alphanumeric characters (`isalnum()`), converts them to lowercase, and joins them into a new string.
2. **Palindrome Check**: `cleaned == cleaned[::-1]` compares the processed string with its reverse. If they match, it's a palindrome.
3. **Unit Tests**: The `unittest` class contains three distinct test methods covering:
   - A classic real-world palindrome with mixed casing and punctuation
   - A straightforward non-palindrome
   - Edge cases (empty string and purely non-alphanumeric input, both correctly return `True`)

### To run the tests:
Save the code in a file (e.g., `palindrome_test.py`) and execute it in your terminal:
```bash
python palindrome_test.py
```
You should see output indicating all 3 tests passed.
