# 27B_HauhauCS_Balanced — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Change your working directory to the top-level folder of the Git repository.
2. **Choose the right tool**: Use `git grep` instead of standard `grep`. It only searches tracked files, automatically respects `.gitignore`, skips binary files, and is highly optimized for Git repositories.
3. **Define search parameters**:
   - Use case-insensitive matching to catch `TODO`, `todo`, `Todo`, etc.
   - Enable line numbers for precise navigation.
   - Rely on default behavior to include filenames in the output.
4. **Execute the search**: Run the command to output all matches in the standard `filepath:line_number:context` format.
5. **Review & process**: Scan the results, optionally redirect to a file (`> todos.txt`) or pipe to other tools for aggregation or tracking.

### Single Bash Command
```bash
git grep -in 'TODO'
```

### Why This Works
| Flag | Purpose |
|------|---------|
| `git grep` | Searches only tracked files, respects `.gitignore`, ignores binaries, and is faster than recursive `grep` |
| `-i` | Case-insensitive matching |
| `-n` | Prints line numbers |
| `'TODO'` | Literal search pattern |

### 💡 Pro Tips
- **Exclude directories**: `git grep -in 'TODO' -- ':!node_modules' ':!vendor' ':!.venv'`
- **Match whole words only** (avoids `TODO_LIST` or `AUTODO`): `git grep -inw 'TODO'`
- **Restrict to specific languages**: `git grep -in 'TODO' -- '*.py' '*.js' '*.ts' '*.c' '*.java' '*.go' '*.rs'`
- **Include untracked files**: `git grep -in 'TODO' -- .` (still tracked) or fall back to `grep -rin --include="*.*" 'TODO' .`
