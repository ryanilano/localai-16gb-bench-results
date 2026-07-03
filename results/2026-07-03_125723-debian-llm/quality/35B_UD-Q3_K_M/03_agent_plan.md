# 35B_UD-Q3_K_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan

1. **Identify Target Files**: Use Git to list all tracked files in the repository. This ensures you only scan version-controlled code, avoiding build artifacts, caches, or unrelated files.
2. **Define Comment Delimiters**: Recognize that different programming languages use different comment syntaxes. Cover the most common ones: `#` (Shell/Python/Ruby), `//` (C/JS/Go/Java), `/* */` (C/Java/JS), `<!-- -->` (HTML/XML/Markdown), `;` (SQL/Ada), and `'` (Python single-line docstrings).
3. **Search for `TODO` After Delimiters**: Use a recursive text search to find lines where `TODO` appears immediately after a comment delimiter. This filters out `TODO` mentions in strings, variable names, or code logic.
4. **Apply Case-Insensitive Matching**: Use case-insensitive search to catch `todo`, `Todo`, `TODO`, etc., since developers rarely follow a strict casing convention.
5. **Format & Output Results**: Ensure the output includes the file path and line number for easy navigation. Sort or deduplicate if necessary (usually not required with `grep`).
6. **Acknowledge Regex Limitations**: Note that pure regex cannot perfectly distinguish comments from multi-line strings or docstrings without a language-aware parser. This approach covers ~95% of real-world cases.

---

### Single Bash Command

```bash
git grep -i -E '(//|#|/\*|<!--|;|'')\s*TODO'
```

---

### How It Works

| Part | Explanation |
|------|-------------|
| `git grep` | Searches only tracked files in the Git repository. Handles filenames with spaces/newlines safely. |
| `-i` | Case-insensitive match (`TODO`, `todo`, `Todo` all match). |
| `-E` | Enables Extended Regular Expressions for cleaner alternation (`|`). |
| `(//|#|/\*|<!--|;|'')` | Matches common comment starters across major languages. |
| `\s*TODO` | Matches zero or more whitespace characters followed by `TODO`. |
| **Output** | Defaults to `filename:line_number:matching_line`, making it easy to jump to each TODO. |

---

### Important Notes
- **Untracked Files**: If you need to scan untracked files too, replace `git grep` with `grep -rn -i -E '(//|#|/\*|<!--|;|'')\s*TODO' .`
- **Multi-line Strings/Docstrings**: Single-line regex cannot perfectly parse `""" ... """` or `/* ... */` that span multiple lines. For 100% accuracy, use a language-aware tool like `ripgrep` (`rg -i '(//|#|/\*|<!--|;|'')\s*TODO'`) or a linter/plugin.
- **False Positives**: Rare cases like `x = "TODO // fix"` may trigger. Manually verify if your codebase heavily uses inline comments inside strings.
