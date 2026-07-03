# 27B_IQ4_XS — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("ls -la")`
2. `run("find . -maxdepth 2 -type f \\( -name '*.yml' -o -name '*.yaml' \\) | head -20")`
3. `read_file(".github/workflows/ci.yml")`
4. `run("python --version && pip --version")`
5. `run("cat pyproject.toml 2>/dev/null || cat setup.py 2>/dev/null || cat requirements.txt 2>/dev/null")`
6. `run("pytest -v")`
7. `run("grep -E 'python-version|os:|install:|run:' .github/workflows/ci.yml")`
8. `run("docker run --rm -v $(pwd):/app -w /app python:3.10-slim bash -c 'pip install -q -e . && pytest -v'")`
9. `run("pytest -v --tb=long -x")`
10. `run("env | grep -iE 'py|path|locale|lang|timezone|http|proxy'")`

**Most likely root-cause hypothesis:** Unpinned dependencies causing different resolved package versions between the local development environment and the CI environment.

**Confirmation tool call:** `run("pip list --format=freeze > local.txt && docker run --rm -v $(pwd):/app -w /app python:3.10-slim bash -c 'pip install -q -e . && pip list --format=freeze' > ci.txt && diff local.txt ci.txt")`
