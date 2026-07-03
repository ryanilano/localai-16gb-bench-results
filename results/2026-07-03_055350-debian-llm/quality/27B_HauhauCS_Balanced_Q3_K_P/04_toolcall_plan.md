# 27B_HauhauCS_Balanced_Q3_K_P — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("ls -la")`
2. `run("find . -maxdepth 3 -type f \\( -name '*.yml' -o -name '*.yaml' -o -name 'Makefile' -o -name 'pyproject.toml' -o -name 'tox.ini' \\) 2>/dev/null")`
3. `read_file(".github/workflows/ci.yml")`
4. `run("python --version && pip list")`
5. `run("pytest --collect-only -q")`
6. `run("docker run --rm -v $(pwd):/app -w /app python:3.9-slim pytest -xvs")`
7. `run("pytest -xvs --tb=long -p no:cacheprovider")`
8. `run("env | sort")`

**Most likely root-cause hypothesis:** The test relies on local environment variables, working directory assumptions, or untracked dependencies that differ from the CI environment (e.g., missing `PYTHONPATH`, relative path resolution, or CI-specific config/secrets).

**One tool call to confirm it:** `run("env -i pytest -xvs")`
