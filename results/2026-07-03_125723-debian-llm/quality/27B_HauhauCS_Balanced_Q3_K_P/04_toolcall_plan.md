# 27B_HauhauCS_Balanced_Q3_K_P — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("ls -la")`
2. `run("find . -maxdepth 2 -type f -name '*.yml' -o -name '*.yaml' -o -name 'Makefile' -o -name 'tox.ini' -o -name 'pytest.ini' -o -name 'pyproject.toml' | head -30")`
3. `read_file(".github/workflows/ci.yml")`
4. `run("python --version")`
5. `run("cat requirements.txt 2>/dev/null || cat pyproject.toml 2>/dev/null || cat setup.cfg 2>/dev/null")`
6. `run("python -m pytest -v")`
7. `run("env | sort")`
8. `run("CI=true python -m pytest -v")`
9. `run("PYTHONPATH=. python -m pytest -v")`
10. `run("python -m pytest -v -x --tb=long 2>&1 | tee ci_debug.log")`
11. `read_file("ci_debug.log")`
12. `run("python -c \"import sys; print('sys.executable:', sys.executable); print('sys.path:', sys.path)\"")`
13. `run("locale")`
14. `run("python -m pytest -v --collect-only")`

**Most likely root-cause hypothesis:** Missing or mismatched dependencies in the CI environment (the local virtual environment contains extra packages or different versions that the tests implicitly depend on, or CI failed to install test dependencies).

**Confirming tool call:** `run("pip list --format=freeze")`
