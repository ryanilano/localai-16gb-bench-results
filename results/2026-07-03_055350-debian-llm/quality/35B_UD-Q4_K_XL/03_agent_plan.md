# 35B_UD-Q4_K_XL — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan

1. **Navigate to the repository root**: Open your terminal and `cd` into the top-level directory of the Git repository.
2. **Choose search scope**: Decide whether to search only version-controlled (tracked) files, or include untracked files as well.
3. **Use Git's optimized search tool**: Run `git grep` instead of standard `grep`. It leverages Git's internal index, making it significantly faster and safer for large repos.
4. **Configure flags for readability**: Add flags to show line numbers, ignore case (to catch `todo`, `TODO`, `Todo`, etc.), and cleanly separate options from the search pattern.
5. **Execute and review**: Run the command. For large output, pipe to `less` or redirect to a file. Review the results to locate each `TODO` with its exact file path and line number.

### Single Bash Command

```bash
git grep -n -i 'TODO'
```

### Flag Breakdown
| Flag | Meaning |
|------|---------|
| `git grep` | Git's built-in, index-optimized text search |
| `-n` / `--line-number` | Prints the line number where the match occurs |
| `-i` / `--ignore-case` | Matches `TODO`, `todo`, `Todo`, etc. |
| `'TODO'` | The literal string to search for |

### Useful Variations
- **Include untracked files**: `git grep -n -i 'TODO' --no-index`
- **Treat binary files as text** (prevents `Binary file matches` warnings): `git grep -n -i -a 'TODO'`
- **Paginate large output**: `git grep -n -i 'TODO' | less`
- **Search only specific languages**: `git grep -n -i 'TODO' -- '*.py' '*.js' '*.go' '*.rs'`

Run from the repo root for consistent results.
