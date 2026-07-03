# 27B_HauhauCS_Balanced — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run("ls -la")`
2. `run("find . -maxdepth 2 -type f \\( -name '*.yml' -o -name '*.yaml' -o -name 'Makefile' -o -name 'tox.ini' -o -name 'pyproject.toml' \\) | sort")`
3. `read_file(".github/workflows/ci.yml")`
4. `run("python --version && python -m pytest --version && pip list")`
5. `run("python -m pytest -x --tb=short 2>&1 | tee local_test.log")`
6. `run("CI_CMD=$(grep -oE 'pytest.*|tox.*|make test' .github/workflows/*.yml .gitlab-ci.yml Makefile 2>/dev/null | head -1) && echo \"$CI_CMD\"")`
7. `run("docker run --rm -v $(pwd):/app -w /app python:3.10-slim bash -c 'pip install -e .[test] && $CI_CMD' 2>&1 | tee ci_repro.log")`
8. `run("grep -B 10 'FAILED\\|ERROR' ci_repro.log | head -50")`
9. `run("python -m pytest $(grep -o 'tests/[^ ]*' ci_repro.log | head -1) -xvs 2>&1 | tee isolated.log")`
10. `run("env | sort > /tmp/local_env.txt && docker run --rm python:3.10-slim env | sort > /tmp/ci_env.txt && diff /tmp/local_env.txt /tmp/ci_env.txt | head -50")`

**Most likely root-cause hypothesis:** Unpinned dependencies or differing Python/runtime environment causing a package version mismatch between local and CI.

**One tool call to confirm it:**
`run("pip freeze > /tmp/local_deps.txt && docker run --rm -v $(pwd):/app -w /app python:3.10-slim bash -c 'pip install -e .[test] && pip freeze' > /tmp/ci_deps.txt && diff /tmp/local_deps.txt /tmp/ci_deps.txt")`
