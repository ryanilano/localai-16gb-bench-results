# 27B_Heretic_NEO_CODE_IQ4_XS — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. run(cmd="ls -la")
2. run(cmd="find . -maxdepth 3 -type f \\( -name '*.yml' -o -name '*.yaml' \\) | head -20")
3. read_file(path=".github/workflows/ci.yml")
4. run(cmd="find . -maxdepth 2 -type f \\( -name 'pyproject.toml' -o -name 'setup.cfg' -o -name 'tox.ini' -o -name 'Makefile' -o -name 'pytest.ini' \\) | head -10")
5. read_file(path="pyproject.toml")
6. read_file(path="Makefile")
7. run(cmd="python --version")
8. run(cmd="python -m venv .venv")
9. run(cmd="source .venv/bin/activate && pip install --upgrade pip")
10. run(cmd="source .venv/bin/activate && pip install -e '.[test]'")
11. run(cmd="source .venv/bin/activate && CI=1 pytest -x -ra -v --tb=short")
12. run(cmd="source .venv/bin/activate && pytest -x -ra -v --tb=short -p no:xdist -p no:randomly")
13. run(cmd="source .venv/bin/activate && PYTHONWARNINGS=all pytest -x -ra -v --tb=short")
14. run(cmd="source .venv/bin/activate && pytest -x -ra -v --tb=short -k 'test_that_fails_in_ci'")

Hypothesis: Non-deterministic test behavior caused by parallel execution (e.g., pytest-xdist) or race conditions in CI, leading to flaky failures that don’t appear in local sequential runs.

Confirming tool call:
run(cmd="source .venv/bin/activate && pytest -x -ra -v --tb=short -p no:xdist -p no:randomly")
