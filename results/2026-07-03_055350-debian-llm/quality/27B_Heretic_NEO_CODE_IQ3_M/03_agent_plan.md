# 27B_Heretic_NEO_CODE_IQ3_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

Step-by-step plan:
1. Open a terminal and change to the root directory of the Git repository.
2. Confirm you’re in a valid Git repo: `git rev-parse --is-inside-work-tree`
3. Use a Git-aware search to scan only tracked files (faster, respects .gitignore, skips .git).
4. Search for TODO-style comments:
   - Case-insensitive (catches TODO, Todo, todo, etc.).
   - Whole-word match only (avoid hitting words like “introduce” or “todomvc”).
   - Exclude binary files (Git handles this automatically).
5. Include file paths and line numbers in the output so each match is easy to navigate.
6. (Optional) Add exclude patterns if certain directories should be skipped (e.g., node_modules, build dirs).

Single bash command:
`git grep -n -i -E "\bTODO\b"`

Brief flag explanation:
- `git grep`: searches only tracked files.
- `-n`: shows line numbers.
- `-i`: case-insensitive.
- `-E "\bTODO\b"`: matches the whole word “TODO” only.

If you also need untracked/ignored files, use:
`grep -rnw -i -E "\bTODO\b" --exclude-dir=".git" .`
