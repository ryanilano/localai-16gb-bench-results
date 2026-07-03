# 27B_HauhauCS_Balanced — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("ls -la")`
2. `run("find . -maxdepth 3 -type f | grep -E '\\.(yml|toml|cfg|ini)$|Makefile' | head -20")`
3. `run("cat .github/workflows/*.yml 2>/dev/null || cat .gitlab-ci.yml 2>/dev/null || cat .travis.yml 2>/dev/null || cat ci.yml 2>/dev/null || echo 'No CI config found'")`
4. `run("python --version && pip list --format=freeze")`
5. `run("python -m pytest -v 2>&1 | tail -50")`
6. `run("env | grep -iE 'python|path|locale|tz|ci|github|action|container' | sort")`
7. `run("python -m pytest -v --tb=long --capture=no 2>&1 | tee /tmp/test_debug.log")`
8. `run("grep -E 'FAILED|ERROR|AssertionError|Timeout|locale|timezone|env|CI' /tmp/test_debug.log | head -50")`
9. `run("cat pyproject.toml 2>/dev/null || cat setup.cfg 2>/dev/null || cat pytest.ini 2>/dev/null")`
10. `run("python -c \"import sys; print('\\n'.join(sys.path))\"")`

**Most likely root-cause hypothesis:** Environment variable differences (e.g., `CI=true`, `GITHUB_ACTIONS=true`, `PYTHONPATH`, or missing configuration variables) triggering different code paths, fallback defaults, or conditional test skips in the CI environment.

**Confirmation tool call:** `run("CI=true GITHUB_ACTIONS=true python -m pytest -v 2>&1 | tail -20")`
