# 35B_UD-Q3_K_M — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

**Plan:**

1.  `run("ls -la")`
2.  `run("find . -type f \( -name '.gitlab-ci.yml' -o -name 'Jenkinsfile' -o -path './.github/workflows/*.yml' \)")`
3.  `run("cat .github/workflows/ci.yml")`
4.  `run("cat .gitlab-ci.yml")`
5.  `run("cat Jenkinsfile")`
6.  `run("cat pyproject.toml")`
7.  `run("cat setup.cfg")`
8.  `run("cat pytest.ini")`
9.  `run("python -m pytest -v")`

**Most likely root-cause hypothesis:**
**Environment Variable Dependency.** The test suite or application code relies on a specific environment variable (e.g., `DB_URL`, `API_KEY`) or a configuration file (e.g., `.env`) that is present in the local environment but is missing or different in the CI environment (often because it is in `.gitignore`).

**Tool call to confirm:**
`run("env")`
