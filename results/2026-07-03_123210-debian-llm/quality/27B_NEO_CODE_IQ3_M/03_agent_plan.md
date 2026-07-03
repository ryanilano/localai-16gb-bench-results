# 27B_NEO_CODE_IQ3_M — 03_agent_plan

## Prompt

You are an agent in a sandbox. Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it.

## Response

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

Here's a thinking process:

1.  **Understand User Request:**
   - **Role:** Agent in a sandbox
   - **Task 1:** Provide a step-by-step plan to find every TODO comment in a Git repository
   - **Task 2:** Provide a single bash command that accomplishes this
   - **Constraints:** Keep it concise, accurate, and practical for a typical Git repo

2.  **Deconstruct Task 1: Step-by-Step Plan**
   What does it take to find all TODO comments in a Git repo?
   - Navigate to the repository directory
   - Identify which files/directories to search (exclude `.git`, `node_modules`, `__pycache__`, build artifacts, etc.)
   - Use a search tool (like `grep`) to find lines containing "TODO" (case-insensitive or exact, usually case-sensitive is fine, but `TODO` is standard)
   - Handle binary files appropriately (skip them)
   - Output results in a readable format (file:line:content)
   - Optionally, verify the results

   Step-by-step plan structure:
   1. Navigate to the root of the Git repository
   2. Use a recursive search tool (like `grep`) to scan all tracked files
   3. Exclude version control directories (`.git`) and common build/cache directories
   4. Filter for lines containing "TODO" (case-sensitive or insensitive as needed)
   5. Output results with file names and line numbers for easy navigation

3.  **Deconstruct Task 2: Single Bash Command**
   Need a robust, single-line bash command that does this.
   Common approach: `grep -r --include="*" -n "TODO" . --exclude-dir=.git --exclude-dir=node_modules --exclude-dir=__pycache__ --exclude-dir=.next --exclude-dir=dist --exclude-dir=build`
   But better to use `git grep` since it's a Git repo. `git grep` automatically handles `.gitignore`, ignores `.git`, and only searches tracked files.
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!__pycache__' ':!.git'`
   Actually, `git grep` ignores `.git` by default. It also respects `.gitignore` if files are tracked. For untracked files, we might want to ignore them anyway.
   Let's stick to `git grep -n "TODO"` as the core. To make it robust and exclude common non-source directories, we can add `-- '*.ts' '*.js' '*.py' '*.java'` etc., but that's too restrictive. Better to just use `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__'`
   Actually, `git grep` is perfect: `git grep -n "TODO"`
   But the user might want a pure bash command that works even if they prefer `grep`. I'll provide a `grep`-based one as it's more universal, but mention `git grep` as the Git-native alternative. Wait, the prompt says "in a Git repo", so `git grep` is actually ideal. I'll stick with `grep` for maximum compatibility, but I'll make it clean:
   `grep -rnw --include="*" -e "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache}`
   Actually, `grep -rn "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} | grep -vE '\.(bin|png|jpg|gif|woff|ttf|eot|svg|pdf|zip|tar|gz)$'` is getting messy.

   Let's use the standard, clean approach:
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next'`
   This is Git-specific, safe, fast, and ignores untracked/binary files by default.

   Alternatively, a pure bash/grep one-liner:
   `grep -rnw --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} -e "TODO" .`

   I think `git grep -n "TODO"` is actually the best answer for a Git repo. I'll provide that, but also explain how to exclude common directories if needed. The prompt asks for "a single bash command that does it". I'll give a robust one:
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`

   Wait, `git grep` doesn't need `--exclude-dir` syntax, it uses pathspecs. The above is correct.
   Actually, simpler: `grep -rn "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} | grep -vE '\.(bin|exe|dll|so|dylib|png|jpg|gif|ico|woff|ttf|eot|svg|pdf|zip|tar|gz|jar|class)$'`
   That's too complex.

   Let's stick to the most practical, widely-used command:
   `git grep -n "TODO"`
   It's literally designed for this. I'll add a note about exclusions if needed, but keep the command simple. Actually, I'll provide a slightly more robust version that handles case-insensitivity and excludes common binary/vendor dirs using `grep` since it's more universally understood, OR just stick to `git grep` and explain.

   I think the best single command is:
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`

   But wait, `git grep` automatically ignores `.git` and respects `.gitignore`. It's perfect.

   Let's draft the step-by-step plan first, then the command.

   **Step-by-step plan:**
   1. Open a terminal and navigate to the root directory of the Git repository.
   2. Identify the search scope: focus on tracked source files while excluding version control metadata (`.git`), build artifacts, dependencies (`node_modules`, `vendor`, etc.), and binary files.
   3. Use a recursive search tool that supports regular expressions and line numbering.
   4. Execute a case-sensitive search for the exact string `TODO` (or case-insensitive if needed) across all files.
   5. Parse the output to get `filename:line_number:matching_line` for quick navigation in an editor.
   6. (Optional) Refine results by excluding specific directories or file extensions if false positives appear.

   **Single bash command:**
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`

   Wait, is there a simpler/more standard way? Many developers just use:
   `grep -rn "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache}`
   But `git grep` is better for Git repos. I'll stick with `git grep` but also provide a pure `grep` alternative if they prefer, but the prompt asks for exactly one command. I'll give the `git grep` one as it's idiomatic.

   Actually, let's make it robust and self-contained:
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`

   I should verify `git grep` syntax for exclusions. Yes, `git grep pattern -- ':!dir'` works. Alternatively, `git grep -n "TODO" -- . ':(exclude)node_modules' ':(exclude)dist' ...` but the `:!` syntax is standard.

   Actually, an even cleaner approach that works universally without pathspec quirks:
   `git grep -n "TODO" | grep -vE '^node_modules|^dist/|^build/|^vendor/|^target/|^__pycache__/|^\.next/|^\.cache/'`
   No, that's messy.

   Let's just use the straightforward `grep` command that works in any Unix-like environment, as it's more predictable:
   `grep -rnw --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} -e "TODO" .`
   Wait, `-w` matches whole words, which might miss `// TODO: fix this`. Better to drop `-w`.
   `grep -rn "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} --exclude="*.bin" --exclude="*.png" --exclude="*.jpg" --exclude="*.gif" --exclude="*.woff" --exclude="*.ttf" --exclude="*.eot" --exclude="*.svg" --exclude="*.pdf" --exclude="*.zip" --exclude="*.tar" --exclude="*.gz" --exclude="*.jar" --exclude="*.class" --exclude="*.pyc" --exclude="*.o" --exclude="*.a" --exclude="*.so" --exclude="*.dylib"`
   Too long.

   I'll stick to `git grep -n "TODO"`. It's the standard for Git repos. I'll add a brief note about exclusions if needed, but keep the command clean. Actually, I'll provide the `git grep` command with common exclusions using `--exclude-dir` equivalent via pathspecs, but I'll simplify it to just `git grep -n "TODO"` and mention that `git grep` automatically skips `.git` and binary files, and respects `.gitignore`. For maximum robustness in one line:

   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`

   Let's verify pathspec syntax: `git grep -n "TODO" -- ':!node_modules' ':!dist'` works in modern git. Alternatively, `git grep -n "TODO" $(git ls-files | grep -vE 'node_modules|dist|build|vendor|target|__pycache__|\.next|\.cache' | xargs -I{} echo ":!:{}")` - no, too complex.

   I'll go with the clean, widely compatible `grep` approach but make it concise:
   `grep -rn "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} --include="*.{js,ts,jsx,tsx,py,java,go,rs,rb,php,c,cpp,h,hpp,cpp,hpp,css,scss,html,md,yml,yaml,json,xml,sh,bash,zsh,sql,sql,tf,hcl,ini,cfg,conf,toml,lua,cs,vb,fs,scala,kt,kts,swift,pl,pm,t,tcl,r,R,ipynb,ex,exs,erl,hrl,hs,ml,mli,clj,cljs,lisp,scheme,rkt,vim,vimrc,zshrc,bashrc,env,envrc,dockerfile,Dockerfile,makefile,Makefile,cmake,CMakeLists,gitignore,gitattributes,editorconfig,prettierrc,eslintrc,babelrc,jsconfig,tsconfig,json,html,htm,xml,xsl,xslt,css,scss,sass,less,styl,stylus,pcss,sss,go,mod,sum,rs,rlib,rlib,rlib,rlib,rlib}"` - no, this is ridiculous.

   Let's step back. The standard, professional answer is:
   **Plan:**
   1. Navigate to repo root
   2. Use a recursive search tool
   3. Exclude `.git`, build dirs, and binary/vendor dirs
   4. Search for "TODO" with line numbers
   5. Output results

   **Command:**
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`

   Actually, `git grep` doesn't need `--exclude-dir` syntax. The `:!` pathspec works. But to be absolutely safe and standard across all git versions, I'll use:
   `git grep -n "TODO" | grep -vE '^(node_modules|dist|build|vendor|target|__pycache__|\.next|\.cache)/'`
   No, `git grep` output format is `file:line:content`, so regex would be `^(node_modules|...)`.

   Honestly, the simplest and most robust single command that works in 99% of cases is:
   `grep -rn "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} --exclude="*.bin" --exclude="*.png" --exclude="*.jpg" --exclude="*.gif" --exclude="*.woff" --exclude="*.ttf" --exclude="*.eot" --exclude="*.svg" --exclude="*.pdf" --exclude="*.zip" --exclude="*.tar" --exclude="*.gz" --exclude="*.jar" --exclude="*.class" --exclude="*.pyc" --exclude="*.o" --exclude="*.a" --exclude="*.so" --exclude="*.dylib"` -> still too long.

   I'll just give:
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`
   And note that `git grep` automatically skips `.git` and binary files.

   Wait, the prompt says "Give a step-by-step plan to find every TODO comment in a Git repo, then a single bash command that does it."
   I'll structure it clearly.

   **Step-by-step plan:**
   1. **Navigate to the repository root** so relative paths resolve correctly.
   2. **Scope the search** to tracked source files while excluding version control metadata (`.git`), dependency directories (`node_modules`, `vendor`, etc.), build/cached artifacts, and binary files.
   3. **Use a recursive search tool** that outputs filenames and line numbers for easy editor navigation.
   4. **Match the pattern** `TODO` (case-sensitive by default, as it's a conventional marker).
   5. **Review results** to verify context and filter out false positives if necessary.

   **Single bash command:**
   ```bash
   git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'
   ```

   I should add a brief explanation of why this works and note that `git grep` is preferred because it automatically ignores `.git`, skips binary files, and respects `.gitignore`. If they want a pure `grep` fallback, I can mention it, but the prompt asks for exactly one command.

   Let's verify `git grep` pathspec exclusion syntax. According to git docs, `git grep pattern -- ':!dir'` works. Example: `git grep foo -- ':!vendor'`. Yes.
   Alternatively, `git grep -n "TODO" -- . ':(exclude)node_modules' ':(exclude)dist' ...` is also valid.
   Actually, the simplest and most foolproof is:
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`
   This is fine.

   I'll format the response cleanly.
   Structure:
   - Step-by-step plan (numbered)
   - Single bash command (code block)
   - Brief explanation of flags/options
   - Note on case-insensitivity or variations if needed.

   Check for edge cases: What if they want case-insensitive? `TODO` is usually uppercase. I'll stick to exact match. I'll add `-i` as an optional note if needed, but keep the command strict.
   What about comments in different languages? `TODO` is universal.
   The plan and command are solid.

   Final check of the command:
   `git grep -n "TODO" -- ':!node_modules' ':!dist' ':!build' ':!vendor' ':!target' ':!__pycache__' ':!.next' ':!.cache'`
   Works in bash. Uses standard git grep. Outputs `file:line:content`. Excludes common junk. Fast. Git-native.

   I'll write it out. 
   One minor improvement: `git grep` doesn't need `--` if pathspecs follow, but it's safer with `--`. Actually, `git grep pattern -- pathspec` is correct.
   I'll present it clearly.✅
   Output matches response. 
   Proceed. 
   Self-Correction/Refinement during thought: 
   Actually, `git grep` automatically ignores `.git` and binary files. The exclusions are good practice. I'll keep it. 
   Some might prefer `grep -rn "TODO" . --exclude-dir={.git,node_modules,__pycache__,dist,build,vendor,target,.next,.cache} --exclude="*.bin" --exclude="*.png" ...` but it's messy. I'll stick with `git grep`. 
   I'll also mention that if they want case-insensitive, they can add `-i`. 
   All good. 
   Output generation. 
   [Done] 
   *Self-Correction/Verification during drafting*:
   Wait, `git grep` pathspec exclusion syntax `:!<dir>` works, but sometimes it's
