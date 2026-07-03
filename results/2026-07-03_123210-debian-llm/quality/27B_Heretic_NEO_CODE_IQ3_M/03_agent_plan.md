# 27B_Heretic_NEO_CODE_IQ3_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

Here’s a concise plan and the exact command.

Step-by-step plan:
1. Open a terminal and change to the root directory of the Git repository.
2. Confirm you’re inside a Git repo:
   - `git rev-parse --is-inside-work-tree`
3. Use Git’s native search to scan all tracked files for “TODO”.
4. Make the search:
   - Case-insensitive (catches TODO/Todo/todo)
   - Include line numbers so you can jump directly to each occurrence
5. Review the output: each line will show `file_path:line_number:matching_text`.
   - Note: this finds all “TODO” tokens in tracked files; distinguishing true comments from strings/comments in docs requires language-specific parsing and is usually not done in a single command.

Single bash command:
- Run from the repo root:
  - `git grep -n -i "TODO"`

If you ever need to include untracked/ignored files as well, you’d fall back to:
- `grep -rn -i "TODO" --include="*" . --exclude-dir=.git`
But for a Git repository, `git grep -n -i "TODO"` is the standard, reliable choice.
