# 27B_NEO_CODE_IQ3_M — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run(cmd="find . -maxdepth 3 -type f \\( -name 'pyproject.toml' -o -name 'setup.cfg' -o -name 'setup.py' -o -name 'pytest.ini' -o -name 'tox.ini' -o -name 'Makefile' \\)")`
2. `run(cmd="find . -maxdepth 3 -type f -name '*.yml' -o -name '*.yaml' | grep -E '(workflow|ci|gitlab|travis|circle)'")`
3. `read_file(path="./pyproject.toml")`
4. `read_file(path="./.github/workflows/ci.yml")`
5. `run(cmd="python -m pytest --collect-only -q")`
6. `run(cmd="python -m pytest -v --tb=short")`
7. `run(cmd="env | grep -E '^(PYTHON|PATH|CI|GITHUB|TF_|LANG|TZ)'")`
8. `run(cmd="CI=true GITHUB_ACTIONS=true PYTHONPATH= python -m pytest -v --tb=short")`
9. `run(cmd="python -m pytest -v --tb=long --override-ini=\"pythonpath=\" --override-ini=\"addopts=\"")`
10. `run(cmd="python -m venv /tmp/ci-venv && /tmp/ci-venv/bin/pip install -e .[test] && /tmp/ci-venv/bin/python -m pytest -v --tb=short")`

**Most likely root-cause hypothesis:** `PYTHONPATH`/`sys.path` configuration mismatch. CI environments typically run with a clean or strictly controlled `PYTHONPATH`, while local development setups often leak the repository root or `src/` directory into `sys.path` (via IDE configuration, `pip install -e`, or shell aliases). This masks import resolution failures, causes tests to load incorrect module versions, or changes test discovery behavior.

**Confirming tool call:** `run(cmd="PYTHONPATH= python -m pytest -v --tb=short")`
