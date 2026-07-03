# 27B_IQ4_XS — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Ensure your working directory is at the top level of the Git repo.
2. **Verify Git context**: Confirm you're inside a valid Git repository so Git-aware tools can function correctly.
3. **Use Git-native search**: Leverage `git grep` instead of standard `grep` to automatically scan only tracked files, ignoring `.git/`, untracked files, and binary assets.
4. **Match the pattern**: Search for the literal string `TODO`. (Note: This catches TODOs in comments, strings, and variables, which is standard practice. Strict comment-only parsing requires language-specific AST tools.)
5. **Format output**: Enable line number display (`-n`) so you can quickly locate each match in its file.
6. **Clean terminal output**: Disable ANSI color codes (`--color=never`) to ensure the output is safe for logs, scripts, or piping.

### Single Bash Command
```bash
git grep -n --color=never "TODO"
```

### Flag Breakdown
| Flag | Purpose |
|------|---------|
| `-n` | Prefixes each match with the file path and line number |
| `--color=never` | Strips ANSI color codes for clean, script-friendly output |
| `"TODO"` | Exact pattern to match (add `-i` for case-insensitive `todo`/`Todo`/`TODO`) |

💡 *Tip: If you want to match only whole-word `TODO` (avoiding `TODOLIST` or `TODOs`), use `git grep -nw --color=never "TODO"`.*
