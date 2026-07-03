# 35B_UD-Q4_K_XL — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("ls -la")`
2. `run("find . -type f -name '*.yml' -o -name '*.yaml' -o -name 'tox.ini' -o -name 'Makefile' | head -5")`
3. `read_file(".github/workflows/ci.yml")`
4. `read_file("pyproject.toml")`
5. `run("pip install -e .")`
6. `run("pytest -v")`

**Most Likely Root-Cause Hypothesis:**
The test relies on an environment variable, system configuration, or file path that exists or is set to a specific value locally (e.g., `os.environ.get('SECRET_KEY', 'local-default')`) but is missing, empty, or different in the CI environment, causing a logic branch failure or assertion error.

**Confirmation Tool Call:**
`read_file(".github/workflows/ci.yml")`
