# 27B_IQ4_XS — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan
1. **Navigate to the repository root**: Open a terminal and `cd` into the top-level directory of the Git repository.
2. **Verify repository state**: Ensure you're on the correct branch/commit (`git status` or `git branch`) since searches only apply to tracked files in the current state.
3. **Choose a Git-native search tool**: Use `git grep` instead of standard `grep` to automatically limit the search to Git-tracked files, respect `.gitignore`, and handle submodules efficiently.
4. **Configure search behavior**:
   - Enable case-insensitivity to catch `TODO`, `Todo`, `todo`, etc.
   - Request line numbers for quick navigation.
   - Skip binary files to avoid false matches and terminal corruption.
5. **Execute the search**: Run the command and review the `file:line:content` output.
6. **Optional refinement**: If you only want to match language-specific comment syntax (e.g., `// TODO`, `# TODO`, `/* TODO */`), adjust the search pattern accordingly.

### Single Bash Command
```bash
git grep -inI "TODO"
```

**Flag breakdown:**
- `-i` : Case-insensitive matching
- `-n` : Prepend line numbers to output
- `-I` : Skip binary files
- `"TODO"` : The literal string to search for (matches anywhere in tracked files)

*Note: This searches only Git-tracked files. If you also need to scan untracked or ignored files, replace with `grep -rnIi "TODO" .`*
