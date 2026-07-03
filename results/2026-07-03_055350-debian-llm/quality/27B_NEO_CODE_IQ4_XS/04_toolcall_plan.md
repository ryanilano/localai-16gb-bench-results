# 27B_NEO_CODE_IQ4_XS — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run(cmd: "ls -la")`
2. `run(cmd: "find . -maxdepth 2 -type f \\( -name '*.yml' -o -name '*.yaml' -o -name 'tox.ini' -o -name 'pytest.ini' -o -name 'setup.cfg' -o -name 'pyproject.toml' \\)")`
3. `read_file(path: ".github/workflows/ci.yml")`
4. `read_file(path: "pyproject.toml")`
5. `run(cmd: "python --version && pip list")`
6. `run(cmd: "python -m pytest --collect-only -q")`
7. `run(cmd: "python -m pytest -v --tb=long")`
8. `run(cmd: "grep -rn \"os.environ\\|os.getenv\\|sys.platform\\|sys.version_info\\|CI\\|GITHUB\" tests/ src/ 2>/dev/null | head -20")`
9. `run(cmd: "env | grep -iE \"python|path|home|user|lang|locale|ci\"")`
10. `run(cmd: "docker run --rm -v \"$(pwd)\":/app -w /app python:3.11-slim bash -c 'pip install -r requirements.txt && python -m pytest -v'")`

**Most likely root-cause hypothesis:** Non-deterministic test execution order or parallelization in CI causes shared state / fixture interference between tests (CI runners often use `pytest-xdist` or different random seeds, while local runs are typically sequential and deterministic).

**Confirming tool call:** `run(cmd: "python -m pytest -v --tb=long -n auto")`
