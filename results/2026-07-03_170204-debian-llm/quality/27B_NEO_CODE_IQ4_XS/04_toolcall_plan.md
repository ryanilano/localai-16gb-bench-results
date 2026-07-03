# 27B_NEO_CODE_IQ4_XS — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("find . -maxdepth 3 -type f -name '*.yml' -o -name '*.yaml' -o -name 'Makefile' -o -name 'tox.ini' -o -name 'pyproject.toml' 2>/dev/null | head -10")`
2. `read_file(".github/workflows/ci.yml")`
3. `run("python --version")`
4. `run("python -m pytest -v --tb=long 2>&1 | tee local_base.log")`
5. `read_file("local_base.log")`
6. `run("python -m pytest -v -n auto --tb=long 2>&1 | tee local_parallel.log")`
7. `read_file("local_parallel.log")`
8. `run("env | grep -iE 'CI|GITHUB|PYTHON|LANG|LC_|TZ|TMP'")`
9. `run("python -m pytest -v --tb=long -x 2>&1 | tee local_stop.log")`
10. `read_file("local_stop.log")`

**Most likely root-cause hypothesis:** Test interference or shared-state pollution caused by parallel execution (e.g., `pytest-xdist` with `-n auto`) in CI, which runs sequentially by default locally.

**Confirmation tool call:** `run("python -m pytest -v -n 1 --tb=long 2>&1 | tee sequential_verification.log")`
