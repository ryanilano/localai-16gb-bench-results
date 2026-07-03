# 27B_Heretic_NEO_CODE_IQ3_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here’s a concise implementation with three unit tests using Python’s `unittest` framework:

```python
import unittest

def is_palindrome(s: str) -> bool:
    # Keep only alphanumeric characters and normalize to lowercase
    cleaned = "".join(ch.lower() for ch in s if ch.isalnum())
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_with_spaces_and_punctuation(self):
        # Classic example; spaces and punctuation should be ignored
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        # Simple non-palindrome
        self.assertFalse(is_palindrome("hello"))

    def test_mixed_case_alphanumeric(self):
        # Ignores case; "racecar" is a palindrome
        self.assertTrue(is_palindrome("RaceCar"))


if __name__ == "__main__":
    unittest.main()
```

You can run this file directly with:
- `python your_file_name.py`
- or `python -m unittest your_file_name.py` 

If you’d prefer a shorter, `assert`-based version instead of `unittest`, I can show that too.
