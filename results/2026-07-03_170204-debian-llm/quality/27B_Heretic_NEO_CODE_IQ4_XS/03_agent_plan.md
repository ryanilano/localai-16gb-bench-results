# 27B_Heretic_NEO_CODE_IQ4_XS — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

Here’s a concise plan and the command.

Step-by-step plan:
1. Open a terminal and navigate to the root of the Git repository.
2. Confirm it’s a valid repo: `git status`.
3. Restrict the search to Git-tracked files only (ignores .git, build dirs, and untracked junk).
4. Recursively search for the literal string "TODO" and include line numbers for context.
5. (Optional) Use case-insensitive search if you also want "todo", "Todo", etc.

Single bash command:
- Exact match: `git grep -n "TODO"`
- Case-insensitive: `git grep -n -i "TODO"`

(If you must use one literal command as requested, use: `git grep -n "TODO"`)
