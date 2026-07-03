# 27B_NEO_CODE_IQ4_XS — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Open a terminal and `cd` into the top-level directory of the Git repository.
2. **Use Git-aware searching**: Prefer `git grep` over standard `grep` so it automatically scans only tracked files, ignores the `.git/` directory, respects `.gitignore`, and skips binary files.
3. **Set search flags**: 
   - Use `-i` for case-insensitive matching (`TODO`, `todo`, `Todo`, etc.)
   - Use `-n` to include line numbers in the output
4. **Execute the search**: Run the command to scan all tracked text files for the pattern.
5. **Review results**: Output will be formatted as `file_path:line_number:matching_line`, making it easy to jump to each TODO in your editor.

### Single Bash Command
```bash
git grep -i -n "TODO"
```

*Note: This finds the literal text `TODO` in tracked files. Distinguishing actual comments from code that merely contains the word "TODO" requires language-specific parsing, which is typically handled by IDEs or linters rather than a single shell command.*
