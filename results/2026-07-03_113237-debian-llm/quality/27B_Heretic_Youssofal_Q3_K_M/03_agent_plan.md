# 27B_Heretic_Youssofal_Q3_K_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Ensure your working directory is the root of the target Git repository.
2. **Verify repository integrity**: Confirm the directory is a valid Git repository using `git status` or `git rev-parse --git-dir`.
3. **Search tracked files**: Use Git's built-in search tool to scan all tracked files for the literal string `TODO`.
4. **Filter results**: Note that Git's search is purely text-based and will match `TODO` anywhere it appears (including in strings, documentation, or logs), not exclusively in comments. You must manually verify or refine results to exclude false positives.
5. **Iterate if needed**: If false positives occur, adjust the search by excluding directories (`--exclude`), using stricter patterns, or combining with external syntax-aware tools. Git itself cannot distinguish comments from code.

### Single Bash Command
```bash
git grep -n "TODO"
```
