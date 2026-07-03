# 35B_UD-Q3_K_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

### Step-by-Step Plan

1. **Navigate to the repo root**  
   Change into the directory containing the `.git` folder. All Git commands will operate relative to this location.

2. **Use a Git-aware search tool**  
   Prefer `git grep` over standard `grep` or `find`. It automatically respects `.gitignore`, excludes the `.git` directory, and only searches tracked files (and optionally untracked ones).

3. **Filter out binary files**  
   Add a flag to skip binaries (images, compiled objects, etc.) to avoid false positives and performance hits.

4. **Search case-insensitively**  
   TODOs can be written as `TODO`, `todo`, `Todo`, etc. Enable case-insensitive matching.

5. **Include line numbers and file paths**  
   Output format should clearly show `<file>:<line>:<content>` so you can jump directly to the code.

6. **Run the command and review results**  
   The output will list every match. For strict "comment-only" matching, you'd need language-specific regex, but `TODO` is universally recognized across languages and usually suffices.

---

### Single Bash Command

```bash
git grep -I -n -i -- 'TODO'
```

#### Flag Breakdown
| Flag | Purpose |
|------|---------|
| `-I` | Ignore binary files |
| `-n` | Show line numbers |
| `-i` | Case-insensitive matching |
| `--` | Separate options from the search pattern |
| `'TODO'` | Pattern to match |

#### Useful Variations
- **Include untracked files:** `git grep --untracked -I -n -i -- 'TODO'`
- **Search entire Git history:** `git log -p --all -S 'TODO'`
- **Only specific languages:** `git grep -I -n -i --include='*.{js,ts,py,cpp,h,go,rs}' -- 'TODO'`

> 💡 **Note:** This searches the current working tree + index. If you need to find TODOs that were later removed, use the history variant above. For strict comment-only matching (e.g., `// TODO` or `# TODO`), you can extend the pattern with regex: `git grep -I -n -i --regex='[/#;{}]\s*TODO' -- .` (adjust syntax based on your shell/Git version).
