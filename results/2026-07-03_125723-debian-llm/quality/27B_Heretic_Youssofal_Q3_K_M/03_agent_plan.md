# 27B_Heretic_Youssofal_Q3_K_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. Navigate to the root directory of the target Git repository.
2. Verify you are inside a valid Git repository (e.g., run `git status` to confirm).
3. Use Git's built-in scoped search tool to scan all tracked files in the current working tree for the exact string `TODO`.
4. Review the returned output, which will safely list only the file paths, line numbers, and matching content from tracked files.

### Single Bash Command
```bash
git grep "TODO"
```
