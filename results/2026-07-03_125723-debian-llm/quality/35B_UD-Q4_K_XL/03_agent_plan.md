# 35B_UD-Q4_K_XL — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan

1. **Navigate to the repository root**: Ensure your terminal is in the top-level directory of the Git repository.
2. **Use Git's native search tool**: Prefer `git grep` over standard `grep` because it automatically:
   - Only searches tracked files (respects `.gitignore`)
   - Skips binary files by default
   - Respects Git's internal encoding and diff settings
3. **Add line number output**: Include the `-n` flag so each match shows exactly which line it appears on.
4. **Execute the search**: Run the command against the entire working tree (`.`).
5. **Review & refine (if needed)**: The output will be formatted as `path/to/file:line_number:matched_line`. If you need case-insensitive matching or want to include untracked files, adjust flags accordingly (noted below).

### Single Bash Command

```bash
git grep -n "TODO" .
```

### 🔍 Notes & Tweaks
- **Case-insensitive**: Use `git grep -in "TODO" .` to catch `todo`, `Todo`, `tOdO`, etc.
- **Include untracked files**: `git grep` only searches committed/tracked content. To include everything, use `grep -rn "TODO" .` (but loses `.gitignore` and binary-file skipping).
- **Strictly comments?**: Language-specific comment syntax varies wildly. A universal text search is the most reliable single-command approach. If you later need strict comment-only extraction, you'd need a language-specific parser (e.g., `rg -t comment "TODO"` with ripgrep).
- **Quiet mode for counts only**: `git grep -c "TODO" .` outputs `filename:count` per file.
