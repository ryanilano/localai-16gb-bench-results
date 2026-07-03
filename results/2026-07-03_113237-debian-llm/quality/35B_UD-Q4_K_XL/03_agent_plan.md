# 35B_UD-Q4_K_XL — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Ensure your terminal is in the top-level directory of the Git project.
2. **Use Git's native search tool**: Leverage `git grep` instead of standard `grep` or `find`. It's optimized for repositories, automatically excludes `.git/`, respects `.gitignore`, and safely skips binary files.
3. **Configure case-insensitivity**: TODO comments often vary in casing (`TODO`, `todo`, `Todo`). Use a flag to match all variants.
4. **Enable line numbers**: Add a flag to output the line number for each match so you can quickly jump to the exact location.
5. **Execute and review**: Run the command. Output will follow the format `path/to/file:line_number:actual_line`.

### Single Bash Command
```bash
git grep -n -i -- "TODO"
```

### Flag Explanation
| Flag | Purpose |
|------|---------|
| `git grep` | Git's built-in search (faster & safer than `find | grep`) |
| `-n` | Prints the line number for each match |
| `-i` | Case-insensitive matching |
| `--` | Separates options from the search pattern (prevents parsing errors) |
| `"TODO"` | The literal string to search for |

### Important Notes
- **Tracked vs Untracked**: By default, this searches only *tracked* files. To include untracked files as well, append a pathspec:  
  `git grep -n -i -- "TODO" .`
- **Common TODO formats**: If you want to match `TODO:`, `TODO(username)`, or `TODO `, use a regex:  
  `git grep -n -i -E "TODO[:\( ]"`
- **Output parsing**: The results can be piped to `less`, `fzf`, or editor commands (e.g., `git grep -n -i -- "TODO" | xargs -I {} vim {}`) for interactive navigation.
