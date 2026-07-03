# 35B_UD-Q3_K_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan

1. **Navigate to the repo root**: Open a terminal and `cd` into the root directory of the Git repository.
2. **Verify the target branch/commit**: Ensure your working tree or index reflects the branch you want to scan (e.g., `main`, `develop`, or a specific commit).
3. **Use Git-aware search**: Prefer `git grep` over system `grep` so the search only covers tracked files, automatically respecting `.gitignore` and excluding untracked/build artifacts.
4. **Define the search pattern**: Search for `TODO` case-insensitively. Optionally extend to common variants like `FIXME`, `XXX`, or `HACK` using extended regex.
5. **Format output for readability**: Enable line numbers and filename headings so you can quickly jump to each match.
6. **Review & triage**: Scan the output, group by file, and prioritize based on context, age, or contributor tags (e.g., `TODO(username)`).

### Single Bash Command

```bash
git grep -n --heading -i -e 'TODO|FIXME|XXX|HACK'
```

### Flag Breakdown
| Flag | Purpose |
|------|---------|
| `git grep` | Searches only tracked files in the index/work tree |
| `-n` | Shows line numbers for each match |
| `--heading` | Prints the filename as a header above each block of matches |
| `-i` | Case-insensitive matching (catches `todo`, `Todo`, etc.) |
| `-e '...'` | Specifies the extended regex pattern to match common TODO variants |

### Optional Tweaks
- **Include untracked files**: Append `--untracked` to the command.
- **Strictly comment-only matches**: Pipe to a second grep if you only want lines starting with comment syntax:
  ```bash
  git grep -n --heading -i -e 'TODO|FIXME|XXX|HACK' | grep -E '^\S+:\d+:[[:space:]]*([#/]|<!--|/\*|::)'
  ```
- **Sort by file**: Append `| sort` to group matches by filename.
