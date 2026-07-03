# 27B_HauhauCS_Balanced — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Ensure your terminal is positioned at the top-level directory of the Git repo.
2. **Scope the search to tracked files**: Use Git's built-in search to automatically exclude the `.git` directory, respect `.gitignore`, and skip binary/build artifacts.
3. **Define the match pattern**: Search for the exact string `TODO`. Use case-insensitivity to catch variations like `todo`, `Todo`, or `TODO`.
4. **Add line context**: Enable line numbers so each result includes the exact location (`file:line:content`) for quick navigation.
5. **Execute & review**: Run the command, parse the output, and address each TODO as needed.

### Single Bash Command
```bash
git grep -n -i "TODO"
```

**Flag breakdown:**
- `git grep`: Native Git search that only scans tracked files, automatically ignores `.git`, and respects `.gitignore`.
- `-n`: Prefixes each match with its line number.
- `-i`: Makes the search case-insensitive.
- `"TODO"`: The pattern to match.

*Optional:* Add `--untracked` to include untracked files, or `-w` to match only whole-word occurrences (e.g., `git grep -n -i -w "TODO"`).
