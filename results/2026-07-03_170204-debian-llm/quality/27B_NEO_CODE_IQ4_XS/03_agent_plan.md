# 27B_NEO_CODE_IQ4_XS — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Open your terminal and `cd` into the top-level directory of the Git repository.
2. **Choose a Git-aware search tool**: Use `git grep` instead of standard `grep` because it automatically respects `.gitignore`, skips the `.git/` directory, and only searches tracked files (avoiding false positives in binaries or build artifacts).
3. **Define the search pattern**: Target the string `TODO`. Enable case-insensitivity to catch `todo`, `Todo`, `TODO`, etc.
4. **Request context**: Include line numbers so you can quickly jump to each match in your editor.
5. **Ensure clean output**: Disable ANSI color codes so the results are easy to read, pipe, or log.
6. **Execute and review**: Run the command and scan the output for file paths, line numbers, and the matching comment lines.

### Single Bash Command
```bash
git grep -n -i "TODO" --color=never
```

### Flag Breakdown
| Flag | Purpose |
|------|---------|
| `git grep` | Searches tracked files in the repo, automatically ignoring `.git/` and binary files |
| `-n` | Prepends the line number to each match |
| `-i` | Case-insensitive matching (`TODO`, `todo`, `Todo`, etc.) |
| `--color=never` | Strips ANSI color codes for clean, script-friendly output |

💡 **Note**: This searches **tracked** files. If you also want to include untracked/new files, use:
```bash
grep -rn --include="*.*" -i "TODO" . --exclude-dir=.git
```
