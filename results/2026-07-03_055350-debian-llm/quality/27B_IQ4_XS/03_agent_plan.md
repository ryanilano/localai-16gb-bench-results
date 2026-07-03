# 27B_IQ4_XS — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Ensure your working directory is at the top level of the Git repo.
2. **Verify Git context**: Confirm you're inside a valid repository (`git rev-parse --is-inside-work-tree`).
3. **Define search scope**: 
   - Search only tracked files (ignores `node_modules`, `vendor`, binaries, etc.)
   - Use case-insensitive matching (`todo`, `Todo`, `TODO`)
   - Match whole words only (avoids false positives like `TODOITEM` or `TODOLIST`)
4. **Execute the search**: Run a single command that recursively scans all tracked files and outputs filename, line number, and matching content.
5. **Review & filter (optional)**: Pipe output to `less` or `grep -v` to exclude specific directories or refine matches if needed.

---

### Single Bash Command
```bash
git grep -i -n -w "TODO"
```

---

### Flag Breakdown
| Flag | Purpose |
|------|---------|
| `-i` | Case-insensitive matching |
| `-n` | Prefixes matches with line numbers |
| `-w` | Matches only whole words (prevents partial matches) |
| `git grep` | Natively searches only tracked files, skips binaries, respects `.gitignore`/`.gitattributes`, and is significantly faster than `grep -r` |

### Important Caveat
`git grep` finds the word `TODO` anywhere in tracked files. Distinguishing **comments** from code/strings is language-dependent. If you strictly want comment-only matches, you'll need language-specific regexes, e.g.:
```bash
# JS/Java/C++/Go
git grep -i -n -E "^\s*(//|/\*|\*).*TODO"
# Python/Ruby/Shell
git grep -i -n -E "^\s*#.*TODO"
```
For a quick, repo-wide audit, the single command above is the standard and most reliable approach.
