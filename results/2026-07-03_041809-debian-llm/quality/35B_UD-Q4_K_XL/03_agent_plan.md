# 35B_UD-Q4_K_XL — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

Here's a thinking process:

1.  **Understand User Request:**
   - **Role:** Agent in a sandbox
   - **Task 1:** Provide a step-by-step plan to find every TODO comment in a Git repo
   - **Task 2:** Provide a single bash command that accomplishes this
   - **Context:** Git repository, TODO comments (typically `TODO:`, `TODO`, `FIXME:`, etc., but I'll focus on `TODO` as requested)

2.  **Deconstruct Task 1: Step-by-step Plan**
   - Need to search through all files in a Git repo
   - Filter for TODO comments (usually in code/text files)
   - Handle different file types and extensions
   - Avoid binary files or directories that shouldn't be searched (e.g., `.git`, `node_modules`, etc.)
   - Extract meaningful output (file path, line number, content)
   - Handle edge cases (case sensitivity, different TODO formats)
   - Step-by-step breakdown:
     a. Navigate to the root of the Git repository
     b. Identify all tracked/untracked files (or just use `git ls-files` for tracked, or `find` for all)
     c. Filter out unwanted directories (`.git`, `node_modules`, `vendor`, etc.)
     d. Use a text search tool (`grep`, `ripgrep`, `ag`) to search for "TODO" (case-insensitive, word boundary)
     e. Format output to show file, line number, and matching line
     f. (Optional) Handle different TODO variations (`TODO:`, `TODO `, `TODO/`, etc.)
     g. Save or display results

3.  **Deconstruct Task 2: Single Bash Command**
   - Need a robust, single-line command
   - Should work in a standard Linux/macOS environment
   - Use `git ls-files` to get tracked files, or `find` with exclusions
   - Use `grep` with appropriate flags
   - Flags for `grep`: `-n` (line number), `-i` (case-insensitive), `-E` (extended regex) or just plain string, `--include` for common extensions, `--exclude-dir` for unwanted dirs
   - Better approach: `git grep -n -i 'TODO'` - This is specifically designed for Git repos and respects `.gitignore`
   - Let's verify: `git grep -n -i 'TODO'` searches all tracked files for TODO, shows line numbers, case-insensitive. It's fast, Git-native, and ignores `.gitignore`d files.
   - What about untracked files? The prompt says "in a Git repo", which usually implies tracked files, but I should mention it. I'll stick with `git grep` as it's the most appropriate.
   - Alternative: `find . -not -path './.git/*' -type f -exec grep -n -i 'TODO' {} +` but `git grep` is better.
   - I'll provide `git grep -n -i '\bTODO\b'` or just `git grep -n -i 'TODO'`
   - Let's refine: `git grep -n -i 'TODO'` is simple and effective. To be more precise about comments, I could use `git grep -n -i -E '(//|#|<!--|/\*)\s*TODO'` but that might be too restrictive across languages. The prompt just says "TODO comment", so `git grep -n -i 'TODO'` is standard.
   - I'll add `--` to separate options from patterns if needed, but not strictly necessary.
   - I'll also mention `rg` (ripgrep) as an alternative if available, but stick to standard tools.
   - Final command: `git grep -n -i 'TODO'`

4.  **Structure the Response:**
   - Step-by-step plan first
   - Single bash command second
   - Explain the command
   - Note caveats (tracked vs untracked, case sensitivity, different TODO formats)
   - Keep it concise and sandbox-appropriate

   Draft:
   **Step-by-Step Plan:**
   1. Navigate to the root directory of the Git repository.
   2. Use Git’s built-in search tool to scan all tracked files, respecting `.gitignore` rules.
   3. Configure the search to be case-insensitive and match whole words to avoid false positives (e.g., "ATODOLATE").
   4. Request line numbers and file paths in the output for easy navigation.
   5. (Optional) Filter out binary files and non-code directories to speed up the search.
   6. Review the results, which will list each match with its file path, line number, and the matching line.

   **Single Bash Command:**
   ```bash
   git grep -n -i --word-regexp 'TODO'
   ```
   Wait, `--word-regexp` might not work exactly as expected with `git grep`. Let's check `git grep` man page. It supports `-w` for word match. So `git grep -n -i -w 'TODO'` is better.
   Actually, `git grep -n -i 'TODO'` is fine. I'll stick with that but add `-w` for safety.
   Let's verify: `git grep -n -i -w 'TODO'` works.
   Alternatively, to catch common TODO comment patterns across languages: `git grep -n -i -E '(//|#|<!--|/\*|;|\'|")\s*TODO'` - too complex. I'll keep it simple and note that it searches all text.
   Actually, the most robust single command for a sandbox environment (assuming standard tools) is:
   `git grep -n -i 'TODO'`
   I'll explain it clearly.

   Let's refine the plan to be more precise:
   1. Ensure you're in the repo root.
   2. Use `git grep` to search tracked files (respects `.gitignore`).
   3. Use `-n` for line numbers, `-i` for case-insensitivity, `-w` for word boundaries.
   4. Pipe to `less` or just output directly.
   5. Handle untracked files if needed (add `find` fallback).
   6. Format/parse output if automation is needed.

   Command: `git grep -n -i -w 'TODO'`
   I'll also provide a version that includes untracked files just in case: `git ls-files -o --exclude-standard | xargs grep -n -i -w 'TODO' 2>/dev/null` but that's messy. I'll stick to `git grep` as it's the standard.

   Actually, a more comprehensive single command that covers both tracked and untracked, excludes `.git` and common noise, and works in any POSIX shell:
   `find . -not -path './.git/*' -not -path './node_modules/*' -type f -exec grep -n -i -w 'TODO' {} + 2>/dev/null`
   But `git grep` is superior for Git repos. I'll recommend `git grep` as primary, and explain why.

   Let's craft the response carefully.
   Structure:
   - Step-by-step plan
   - Single bash command
   - Explanation of flags
   - Notes on edge cases (tracked vs untracked, language-specific comment syntax)
   - Keep it tight.

   Self-Correction/Verification during thought:
   - `git grep -w 'TODO'` might miss `TODO:` because `:` is not a word character. `-w` matches word boundaries. `TODO:` has `:` after, which is a non-word char, so `-w` actually works fine for `TODO:`. But `TODO-1` might be split. Usually `TODO` is fine.
   - I'll use `git grep -n -i 'TODO'` to be safe and catch all variations, then mention `-w` as optional.
   - Actually, `git grep -n -i -e 'TODO'` is standard.
   - I'll stick with `git grep -n -i 'TODO'` as the command. It's simple, reliable, and Git-native.
   - I'll make sure the plan explicitly mentions handling different file types and comment syntaxes if needed.

   Final check of the prompt: "find every TODO comment in a Git repo, then a single bash command that does it."
   I'll provide exactly that.

   Plan:
   1. Navigate to repo root
   2. Identify file scope (tracked vs all)
   3. Choose search tool (`git grep` recommended)
   4. Configure flags (case-insensitive, line numbers, word boundary)
   5. Execute and parse output
   6. (Optional) Filter by comment syntax if needed

   Command: `git grep -n -i -w 'TODO'`
   Explanation of flags: `-n` line numbers, `-i` case-insensitive, `-w` whole word
   Note: Respects `.gitignore`, only searches tracked files. For untracked, add fallback.

   I'll write it out clearly.✅
   One minor improvement: To specifically target *comments*, I could suggest `git grep -n
