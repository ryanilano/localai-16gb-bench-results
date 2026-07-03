# 27B_Heretic_Youssofal_Q3_K_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here is the Python code containing the function and the required unit tests using the standard `unittest` library.

```python
import unittest

def is_palindrome(s):
    """
    Checks if a string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter out non-alphanumeric characters and convert to lowercase
    cleaned_s = "".join(char.lower() for char in s if char.isalnum())
    
    # Check if the cleaned string is the same forwards and backwards
    return cleaned_s == cleaned_s[::-1]


# --- Unit Tests ---

class TestPalindrome(unittest.TestCase):
    
    def test_standard_palindrome(self):
        # Test 1: Standard valid palindrome
        self.assertTrue(is_palindrome("racecar"))

    def test_complex_palindrome(self):
        # Test 2: Palindrome with spaces, punctuation, and mixed case
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_not_palindrome(self):
        # Test 3: Non-palindrome string
        self.assertFalse(is_palindrome("hello"))

if __name__ == '__main__':
    unittest.main()
```

### How it works:
1.  **Cleaning**: The list comprehension `[char.lower() for char in s if char.isalnum()]` iterates through the string. It discards any character that is not a letter or number (`isalnum()` check) and converts valid characters to lowercase.
2.  **Checking**: The cleaned string is compared against its reverse (`[::-1]`). If they match, it returns `True`.
3.  **Testing**: The `unittest` class ensures the function handles standard cases, complex formatting, and negative cases correctly.
