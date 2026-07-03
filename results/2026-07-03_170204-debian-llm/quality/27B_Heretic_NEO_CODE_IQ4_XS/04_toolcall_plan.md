# 27B_Heretic_NEO_CODE_IQ4_XS — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. run(cmd: "ls -la")
2. run(cmd: "find . -maxdepth 3 -type f -name '*.yml' | grep -iE '(github|circle|gitlab|bitbucket|azure|jenkins|ci)' | head -1")
3. read_file(path: ".github/workflows/ci.yml")
4. run(cmd: "find . -maxdepth 3 -type f \\( -name 'pyproject.toml' -o -name 'setup.cfg' -o -name 'tox.ini' -o -name 'noxfile.py' -o -name 'Makefile' \\) | head -5")
5. read_file(path: "pyproject.toml")
6. read_file(path: "setup.cfg")
7. run(cmd: "find . -type f -name 'test_*.py' -o -name '*_test.py' | head -20")
8. run(cmd: "python3 --version")
9. run(cmd: "cat requirements.txt || cat requirements-dev.txt || cat requirements-test.txt || echo 'no_requirements'")
10. run(cmd: "pip install -r requirements.txt || pip install -r requirements-dev.txt || pip install -r requirements-test.txt || true")
11. run(cmd: "python -m pytest -v --tb=long 2>&1 | tee ci_failure.log")
12. run(cmd: "CI=1 GITHUB_ACTIONS=true CONTINUOUS_INTEGRATION=true python -m pytest -v --tb=long 2>&1 | tee ci_failure_cienv.log")
13. run(cmd: "python -m pytest -v --tb=long -p no:randomly -p no:flaky 2>&1 | tee ci_failure_deterministic.log")
14. run(cmd: "grep -E '(FAIL|ERROR|AssertionError)' ci_failure*.log | head -20")
15. run(cmd: "python -m pytest $(grep -oP 'tests/.*::.*' ci_failure*.log | head -1) -v --tb=long -s 2>&1 | tee ci_failure_isolated.log")
16. run(cmd: "docker run --rm -v $(pwd):/repo -w /repo python:3.11-slim bash -c 'pip install -r requirements.txt && python -m pytest -v --tb=long' 2>&1 | tee ci_failure_docker.log")

Single most likely root-cause hypothesis:
The test or application code behaves differently under CI-specific environment variables or a stricter/ephemeral environment (e.g., CI=1, GITHUB_ACTIONS, missing local env vars, different locale/timezone, network/permission constraints, or unpinned dependencies), causing failures that do not occur in a developer’s local setup.

One confirming tool call:
run(cmd: "CI=1 GITHUB_ACTIONS=true CONTINUOUS_INTEGRATION=true HOME=/tmp PYTHONPATH='' python -m pytest -v --tb=long 2>&1 | tee ci_hypothesis_test.log")
