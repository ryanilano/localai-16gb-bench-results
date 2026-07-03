# 27B_HauhauCS_Balanced — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("find . -maxdepth 3 -type f \\( -name '*.yml' -o -name '*.yaml' -o -name 'Jenkinsfile' -o -name 'tox.ini' -o -name 'Makefile' \\) | head -20")`
2. `read_file(".github/workflows/ci.yml")`
3. `run("grep -A 3 'run:' .github/workflows/ci.yml | grep -E '(pytest|python -m unittest|tox|make test)'")`
4. `read_file("pyproject.toml")`
5. `run("python -m pytest --version 2>/dev/null || python -m unittest --help 2>/dev/null")`
6. `run("python -m pytest -xvs --tb=long")`
7. `run("python -m venv .ci_venv && source .ci_venv/bin/activate && pip install -e '.[dev]' && python -m pytest -xvs --tb=long")`
8. `run("python -m pytest -xvs --tb=long -n auto")`
9. `run("python -m pytest -xvs --tb=long --forked")`
10. `run("env -i PATH=$PATH HOME=$HOME python -m pytest -xvs --tb=long")`

**Most likely root-cause hypothesis:** Test pollution / order-dependent tests caused by uncleaned global state, module-level caches, or shared fixtures leaking between test cases.

**One tool call to confirm it:** `run("python -m pytest -xvs --tb=long --forked")`
