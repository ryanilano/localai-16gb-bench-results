# 35B_UD-Q4_K_XL — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

Here's a thinking process that leads to the suggested response:

1.  **Analyze the Request:**
    *   **Role:** Autonomous coding agent.
    *   **Tools:** `read_file(path)`, `run(cmd)`, `write_file(path, content)`.
    *   **Task:** Create a plan to reproduce and locate a CI-only failure locally.
    *   **Constraints:**
        *   Express the plan as an *ordered list of concrete tool calls* (no prose).
        *   Assume nothing about repo layout (discover it).
        *   After the plan, provide the *single most likely root-cause hypothesis* and the *one tool call* to confirm it.

2.  **Deconstruct the Problem (CI vs. Local):**
    *   Why do tests fail in CI but pass locally?
        *   Environment differences (OS, Python version, dependencies).
        *   Configuration differences (env vars, config files).
        *   State differences (database state, file system permissions, race conditions).
        *   CI specific infrastructure (network access, mocked services).
    *   *Strategy:*
        1.  Discover the project structure.
        2.  Identify the CI configuration (GitHub Actions, GitLab CI, Jenkins, etc.).
        3.  Identify the test runner configuration (pytest, unittest, tox, etc.).
        4.  Compare local environment with CI environment (env vars, versions).
        5.  Attempt to reproduce the specific failure.

3.  **Drafting the Plan (Step-by-Step Tool Calls):**

    *   *Step 1: Discovery.* I need to see what files exist.
        *   `run("ls -la")`

    *   *Step 2: CI Config.* Look for common CI files.
        *   `run("find . -name '.github' -o -name '.gitlab-ci.yml' -o -name 'Jenkinsfile' -o -name '.circleci' | head -n 5")` (Or just look at the root for common files). Let's stick to finding the CI config file directly.
        *   `run("ls -la .github/workflows/ || ls -la .gitlab-ci.yml || ls -la .circleci/ || ls -la Jenkinsfile || echo 'No common CI config found'")`

    *   *Step 3: Test Config.* How are tests run?
        *   `read_file("pyproject.toml")` (Check for pytest config)
        *   `read_file("setup.cfg")`
        *   `read_file("tox.ini")`
        *   `read_file("pytest.ini")`
        *   Let's check the CI file first to see how it invokes tests, then check the code.
        *   *Refined Plan:*
            1.  `run("ls -la")` (Root structure)
            2.  `run("cat .github/workflows/*.yml 2>/dev/null || cat .gitlab-ci.yml 2>/dev/null || cat .circleci/config.yml 2>/dev/null || echo 'No standard CI config found'")` (Get CI script)
            3.  `run("grep -r 'python' .github/ .gitlab-ci.yml .circleci/ 2>/dev/null | grep -i test")` (Find how tests are triggered)
            4.  `run("pip list 2>/dev/null || cat requirements.txt 2>/dev/null || cat setup.py 2>/dev/null || cat pyproject.toml 2>/dev/null")` (Check dependencies)
            5.  `run("python --version")` (Check local Python version)
            6.  `run("env | sort")` (Check local environment variables)
            7.  `run("git diff HEAD~1 --name-only 2>/dev/null || git log --oneline -1")` (Check recent changes or current state)

    *   *Let's refine the tool calls to be more specific and robust based on the "discover it" instruction.*

    *   *Plan Sequence:*
        1.  List root directory to identify the project type.
        2.  Identify the CI configuration file (GitHub, GitLab, Circle, etc.).
        3.  Read the CI configuration to extract the test command and environment variables.
        4.  Read the dependency file (requirements.txt, pyproject.toml, etc.).
        5.  Check the local Python version and installed packages.
        6.  Check local environment variables (specifically looking for things like `CI=true` or specific secrets).
        7.  Run the test command exactly as defined in the CI config (or a subset of it) to see if it fails.

4.  **Selecting the Root Cause Hypothesis:**
    *   The most common reason for "Passes Local, Fails CI" in Python projects is the difference in environment variables, specifically `PYTHONPATH` or missing secrets/mock configurations, or a specific dependency version mismatch.
    *   However, the *most* classic one is the environment variable `CI=true` or specific configuration toggles that behave differently when running in a container vs. a developer machine. Another strong candidate is file system permissions (read-only in CI) or locale settings.
    *   *Hypothesis:* The test relies on a specific environment variable that is set in the CI configuration but not in the local environment, or vice versa.
    *   *Confirmation:* Compare the local `env` output with the CI config's `env` block.

5.  **Refining the Output Format:**
    *   The user requested: "expressed as an ordered list of concrete tool calls... (not prose)."
    *   I need to map the thought process to the specific tool signatures.

    *   *Drafting the Tool Calls:*
        1.  `run("ls -la")`
        2.  `run("find . -maxdepth 2 -name '.yml' -o -name '.yaml' -o -name 'Jenkinsfile' | grep -E '(ci|workflow|build)'")` (To find CI config)
        3.  `read_file("path/to/ci/config")` (Wait, I don't know the path yet. I need to run the find command first to get the path).
        *   *Correction:* The prompt asks for a plan *as* tool calls. Since I can't see the output of step 1 to determine step 2's arguments dynamically *in the prompt generation phase* (unless I simulate the execution), I should provide a static plan that covers the most likely paths or uses a broad discovery command.
        *   *Actually*, the prompt implies I am generating the *plan* now. I can't execute it yet. I must provide the list of calls I *would* make. Since I don't know the repo layout, I have to assume a standard discovery flow.

    *   *Revised Tool Call List:*
        1.  `run("ls -la")`
        2.  `run("find . -maxdepth 3 -type f \( -name '*.yml' -o -name '*.yaml' -o -name 'Jenkinsfile' -o -name '.gitlab-ci.yml' \)")`
        3.  `read_file("path/to/ci/file.yml")` (I will assume a generic placeholder or specific common ones).
        *   *Better approach:* Use `run` to find the file, then `read_file` on the result. Since I am generating the plan textually, I will list the commands assuming the outputs of previous steps guide the arguments.

        1.  `run("ls -la")`
        2.  `run("find . -maxdepth 3 -type f -name '*.yml' | head -n 10")`
        3.  `read_file("path/to/ci/file.yml")`
        4.  `run("cat path/to/ci/file.yml")` (To see env vars)
        5.  `run("env")` (To see local env)
        6.  `run("python --version")`
        7.  `run("pip list")`
        8.  `run("python -m pytest --collect-only path/to/failing/test.py")` (Hypothetical)

    *   *Let's stick to the prompt's strict constraint:* "ordered list of concrete tool calls with the exact arguments you would pass at each step".
    *   Since I cannot see the output of `ls -la` to know the exact filename of the CI config, I will provide a script-like plan that discovers the config file and then reads it.

    *   *Plan:*
        1.  `run("ls -la")`
        2.  `run("find . -maxdepth 3 -type f \( -name '.gitlab-ci.yml' -o -name 'Jenkinsfile' -o -name '.circleci' -o -name 'azure-pipelines.yml' \) -o -path './.github/workflows/*.yml'")`
        3.  `read_file("path/to/ci/config.yml")`
       
