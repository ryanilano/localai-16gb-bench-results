# 27B_Heretic_Youssofal_Q3_K_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. Navigate to the root directory of the target Git repository.
2. Verify that the directory is a valid Git repository and that you have read access.
3. Use Git's built-in search functionality to scan all tracked files for the exact string `TODO`.
4. Note that this will match any occurrence of `TODO` in the codebase, including strings, identifiers, or documentation, not just actual comments. Automatically distinguishing comments from executable code or string literals is not possible with a single bash command due to syntax differences across all programming languages.
5. Review the output manually to filter out false positives and retain only actual TODO comments.

### Single Bash Command
```bash
git grep -n "TODO"
```
