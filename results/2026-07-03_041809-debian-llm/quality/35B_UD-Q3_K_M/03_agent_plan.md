# 35B_UD-Q3_K_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

Here's a thinking process:

1.  **Understand User Goal**: The user wants to find every `TODO` comment in a Git repository. They want:
   - A step-by-step plan
   - A single bash command that accomplishes it

2.  **Identify Key Components**:
   - Git repository context
   - Search for comments containing "TODO"
   - Handle different comment syntaxes (e.g., `#`, `//`, `/* */`, `<!-- -->`)
   - Work across all files in the repo
   - Output should be clear and useful (file path, line number, content)

3.  **Step-by-Step Plan Development**:
   - Step 1: Navigate to the root of the Git repository (or ensure we're in one).
   - Step 2: Identify all tracked/untracked files in the repo.
   - Step 3: Search through these files for lines containing comment markers followed by "TODO" (case-insensitive usually preferred).
   - Step 4: Filter results to exclude binary files or non-text files if necessary.
   - Step 5: Format the output to show file path, line number, and the matching line.
   - Step 6: Run the command and review results.

   Refined Plan:
   1. Ensure you're in the root of the Git repository.
   2. Use `git ls-files` to get a list of all tracked files.
   3. Use `grep` to search through those files for lines containing comment syntax (`#`, `//`, `/*`, `<!--`, etc.) followed by "TODO".
   4. Use `grep` options for case-insensitivity, line numbers, and recursive/file-list input.
   5. Handle potential issues like binary files, hidden files, and different comment styles.
   6. Combine everything into a single, robust bash command.

4.  **Develop the Single Bash Command**:
   - Need to search for TODOs in comments. Common comment prefixes: `#`, `//`, `/*`, `<!--`, `--`, `;`, `'` (depending on language).
   - Instead of hardcoding all comment styles, a more robust approach is to search for `TODO` and then optionally filter by comment markers, but the user specifically said "TODO comment". However, in practice, searching for `TODO` across all files is often sufficient, but to be precise about *comments*, I should look for common comment delimiters.
   - Actually, the most reliable way in a multi-language repo is to use `git ls-files` piped to `grep` with a regex that matches comment patterns followed by TODO.
   - Regex idea: `(#[^#]*|//[^/]*|/\*|\*/|<!--|-->|--|;|')\s*TODO` - too complex and error-prone.
   - Simpler approach: `grep -rn -i --include='*.{ext}' TODO` but that's not comment-specific.
   - Let's stick to a practical, widely-used approach: Search for `TODO` in all tracked files, but filter to only lines that look like comments. A good heuristic is to look for `TODO` preceded by common comment markers.
   - Actually, the standard tool for this is `git grep` or `grep` with `git ls-files`.
   - Let's craft a command that searches for `TODO` in comment lines. A pragmatic regex: `^[[:space:]]*(#|//|/\*|<!--|--|;|'|<!--).*TODO`
   - Better: Use `grep -iE '(#|//|/\*|<!--|--|;|'')\s*TODO'` but this might miss some.
   - Actually, the most robust single-line command used in practice is:
     `git ls-files | xargs grep -inE '(#|//|/\*|<!--|--|;|'')\s*TODO'`
   - But `xargs` might fail on large repos. `git ls-files -z | xargs -0 grep ...` is safer.
   - Let's refine: `git ls-files -z | xargs -0 grep -inE '(#|//|/\*|<!--|--|;|'')\s*TODO'`
   - Wait, `grep` might complain about binary files. Add `--text` or `-I` (skip binary).
   - Also, `TODO` might be in code, not comments. The prompt says "TODO comment". I'll focus on comment syntax.
   - Let's make it more precise: `grep -inE '^[[:space:]]*(#|//|/\*|<!--|--|;|'')\s*TODO'`
   - Actually, a simpler and more standard approach is just `git grep -i 'TODO' -- '*.txt' '*.md' '*.sh' '*.py' '*.js' '*.c' '*.h'` but that's not comprehensive.
   - I think the best balance of accuracy and simplicity is:
     `git ls-files -z | xargs -0 grep -inE '(#|//|/\*|<!--|--|;|'')\s*TODO' --text`
   - Let's test mentally: `git ls-files -z` outputs null-separated tracked files. `xargs -0` passes them to `grep`. `grep -inE` does case-insensitive extended regex. The pattern matches comment starters followed by TODO. `--text` forces grep to treat binary as text.
   - One issue: Some languages use different comment styles. This covers most common ones.
   - Alternative: Just search for `TODO` and let the user filter, but the prompt specifically says "TODO comment". I'll stick with the comment-aware regex.
   - Actually, a more robust single command used by many devs is:
     `git grep -i 'TODO' --include='*.sh' --include='*.py' --include='*.js' --include='*.c' --include='*.h' --include='*.md' --include='*.txt' --include='*.yml' --include='*.yaml' --include='*.json' --include='*.xml' --include='*.html' --include='*.css' --include='*.scss' --include='*.rb' --include='*.java' --include='*.kt' --include='*.go' --include='*.rs' --include='*.php' --include='*.swift' --include='*.ts' --include='*.jsx' --include='*.tsx' --include='*.vue' --include='*.svelte' --include='*.sql' --include='*.dockerfile' --include='Dockerfile*' --include='Makefile*' --include='CMakeLists*' --include='*.cmake' --include='*.ini' --include='*.cfg' --include='*.conf' --include='*.toml' --include='*.env' --include='*.properties' --include='*.gradle' --include='*.pom' --include='*.xml' --include='*.plist' --include='*.yaml' --include='*.yml' --include='*.json' --include='*.json5' --include='*.toml' --include='*.ini' --include='*.cfg' --include='*.conf' --include='*.env' --include='*.properties' --include='*.gradle' --include='*.pom' --include='*.xml' --include='*.plist' --include='*.yaml' --include='*.yml' --include='*.json' --include='*.json5' --include='*.toml'` -> This is ridiculous.
   - Better: `git ls-files -z | xargs -0 grep -inE '(#|//|/\*|<!--|--|;|'')\s*TODO' --text` is fine.
   - Actually, `git grep` is better: `git grep -iE '(#|//|/\*|<!--|--|;|'')\s*TODO'` but `git grep` doesn't search untracked files by default. The prompt says "in a Git repo", which usually implies tracked files, but I'll note untracked if needed. I'll stick to `git ls-files` for tracked, or just `grep -r` if they want all files. I'll specify tracked files for safety.
   - Let's refine the regex to be more inclusive of comment styles: `^[[:space:]]*(#|//|/\*|<!--|--|;|'')\s*TODO`
   - Actually, a simpler and very effective command is:
     `git ls-files -z | xargs -0 grep -inE '(#|//|/\*|<!--|--|;|'')\s*TODO' --text`
   - I'll add `--no-filename`? No, they need file paths.
   - I'll also add `--color=auto` for readability, but it's optional.
   - Let's structure the plan clearly.

5.  **Draft the Response**:
   - **Step-by-step plan**:
     1. Verify you're in the root of the Git repository.
     2. List all tracked files using `git ls-files`.
     3. Pass the file list to `grep` to search for lines containing common comment delimiters followed by the word "TODO".
     
