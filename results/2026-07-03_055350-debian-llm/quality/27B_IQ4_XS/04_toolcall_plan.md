# 27B_IQ4_XS — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("find . -maxdepth 4 -type f \\( -name '*.yml' -o -name '*.yaml' -o -name 'Dockerfile' -o -name 'Makefile' -o -name 'pyproject.toml' -o -name 'setup.cfg' -o -name 'tox.ini' \\) 2>/dev/null | sort")`
2. `run("cat $(find . -maxdepth 4 -type f -name '*.yml' -o -name '*.yaml' 2>/dev/null | grep -iE '(github|gitlab|circleci|ci)' | head -1)")`
3. `run("grep -E 'run:|script:|command:|pytest|coverage|test' $(find . -maxdepth 4 -type f -name '*.yml' -o -name '*.yaml' 2>/dev/null | grep -iE '(github|gitlab|circleci|ci)' | head -1)")`
4. `run("python --version && pip --version && locale && echo $PATH && echo $HOME")`
5. `run("docker run --rm -v $(pwd):/app -w /app python:3.11-slim bash -c 'pip install -r requirements.txt && pytest -v --tb=long' 2>&1 | tee ci_replica.log")`
6. `run("grep -E 'FAIL|ERROR|AssertionError|ImportError|FileNotFoundError' ci_replica.log")`
7. `run("python -m pytest -v --tb=long 2>&1 | tee local_native.log")`
8. `run("diff ci_replica.log local_native.log | grep -E 'FAIL|ERROR|AssertionError' | head -10")`

**Most Likely Root-Cause Hypothesis:**
Environment/Platform Mismatch: CI runs on a clean Linux runner with a specific Python version, fresh virtual environment, and default `C` locale/timezone, while the local machine uses a different OS, cached/extra dependencies, or non-default locale. This causes platform-specific path handling (`os.sep`, case sensitivity), missing optional dependencies, or locale/timezone-dependent string/datetime comparisons to fail only in CI.

**Confirmation Tool Call:**
`run("diff <(python -c 'import sys, os, locale; print(sys.platform, sys.version, os.sep, locale.getlocale())') <(docker run --rm python:3.11-slim python -c 'import sys, os, locale; print(sys.platform, sys.version, os.sep, locale.getlocale())')")`
