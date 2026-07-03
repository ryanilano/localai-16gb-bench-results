# 27B_Heretic_NEO_CODE_IQ4_XS — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

Here’s a concise, practical approach.

Step-by-step plan:
1. Open a terminal and move to the root of the Git repository:
   - `cd /path/to/repo`
2. Decide the search scope:
   - Tracked files only (recommended, respects .gitignore, faster).
   - Or all files including untracked (if you want everything on disk).
3. Use a recursive search with:
   - Case-insensitive matching (catches TODO, todo, Todo).
   - Line numbers for easy navigation.
   - Exclusion of common noise directories (node_modules, .git, etc.).
4. Run the search and review the output.

Single bash command (tracked files, recommended):
- `git grep -i -n "TODO" -- .`

If you must search all files (tracked + untracked), use:
- `grep -rn -i --exclude-dir={.git,node_modules,vendor,__pycache__,.venv,.mypy_cache,.pytest_cache} "TODO" .`

Notes:
- These commands find lines containing "TODO"; if you strictly need only comment-style TODOs (e.g., `// TODO`, `# TODO`, `/* TODO */`), you’d need language-specific patterns, which usually aren’t worth the complexity unless required.
