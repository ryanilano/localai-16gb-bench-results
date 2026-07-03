# 27B_HauhauCS_Balanced — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Open a terminal and `cd` into the top-level directory of your Git repository.
2. **Verify Git context**: Ensure you're in a valid Git repo (optional but recommended: `git rev-parse --is-inside-work-tree`).
3. **Choose a Git-aware search tool**: Use `git grep` instead of standard `grep` so it automatically skips the `.git/` directory, ignores untracked files, and handles binary files safely.
4. **Search case-insensitively**: Match `TODO`, `todo`, `Todo`, etc., to catch all comment variations.
5. **Capture location context**: Ensure the output includes file paths and line numbers so you can jump directly to each TODO.
6. **Review results**: Scan the output to confirm all TODO comments are captured and address them as needed.

### Single Bash Command
```bash
git grep -i "TODO"
```

### Why This Works
- `git grep` only searches files tracked by Git, making it fast and clean.
- `-i` enables case-insensitive matching.
- Output format is `filepath:line_number:matching_line`, which works seamlessly with editors like VS Code (`:open` or `Ctrl+Click`) or Vim (`:split`).
- *(Optional)* Add `-w` (`git grep -iw "TODO"`) if you want to exclude partial matches like `TODOLIST` or `UNDONE`.
