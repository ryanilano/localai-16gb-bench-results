# 27B_HauhauCS_Balanced_Q3_K_P — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

**Step-by-Step Plan:**
1. Navigate to the root directory of the Git repository.
2. Use Git's built-in search tool (`git grep`) to scan only tracked files, which automatically skips the `.git` directory and respects `.gitignore`.
3. Configure the search to be case-insensitive and to display line numbers for precise location tracking.
4. Execute the command to output a list of every file and line containing "TODO".
5. Review the output to verify all TODO comments are captured.

**Single Bash Command:**
```bash
git grep -n -i "TODO"
```

*Optional refinement:* If you want to strictly match only comment syntax (ignoring TODOs inside string literals or variable names), use this regex-based variant instead:
```bash
git grep -n -i -E '(#|//|/\*|;\s*).*TODO'
```
