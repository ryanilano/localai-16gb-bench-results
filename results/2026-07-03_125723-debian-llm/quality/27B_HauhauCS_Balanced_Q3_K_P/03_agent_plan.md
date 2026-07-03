# 27B_HauhauCS_Balanced_Q3_K_P — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repo root**: Open a terminal and `cd` into the top-level directory of your Git repository.
2. **Verify Git state**: Optionally run `git status` to confirm you're in a valid repository.
3. **Execute the search**: Run the provided bash command to scan all tracked files for TODO markers.
4. **Interpret results**: Output will follow the format `file_path:line_number:matching_line`. Review each match to confirm it's a comment and note the location.
5. **Save/Track (Optional)**: Pipe the output to `less` for scrolling (`| less`) or redirect to a file (`> todos.txt`) for issue tracking or bulk cleanup.

---

### Single Bash Command
```bash
git grep -i "TODO"
```

### Why This Works
- `git grep`: Searches only files tracked by Git (respects `.gitignore` and ignores untracked/build files), making it significantly faster and more accurate than `grep -r`.
- `-i`: Case-insensitive flag to catch `todo`, `Todo`, `TODO`, etc.
- By default, prints `filename:line_number:full_line` for every match.

💡 **Strict Comment Matching (Optional)**  
If you want to exclude TODOs that appear in strings or variable names and only match actual comments, use this regex-enhanced version:
```bash
git grep -iE "(^|.*\s)(//|#|/\*|\*|--|;)\s*TODO"
```
This looks for common comment prefixes (`//`, `#`, `/*`, `*`, `--`, `;`) followed by `TODO`. Adjust the prefix list to match your project's languages.
