# 27B_Heretic_NEO_CODE_IQ3_M — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. run("pwd")
2. run("find . -maxdepth 4 -type f \\( -name 'pyproject.toml' -o -name 'setup.cfg' -o -name 'setup.py' -o -name 'tox.ini' -o -name 'pytest.ini' -o -name 'Makefile' \\) | head -50")
3. run("find . -maxdepth 4 -type f \\( -name '.gitlab-ci.yml' -o -name 'azure-pipelines.yml' -o -name 'Jenkinsfile' \\) -o -type d -name '.github' | head -20")
4. run("find . -path '*/.github/workflows' -type f \\( -name '*.yml' -o -name '*.yaml' \\) | head -20")
5. read_file("pyproject.toml")
6. read_file("setup.cfg")
7. read_file("tox.ini")
8. run("ls .github/workflows/")
9. run("CI_FILE=$(find .github/workflows -type f \\( -name 'ci.yml' -o -name 'test.yml' -o -name 'python.yml' \\) | head -1); cat \"\$CI_FILE\"")
10. run("python3 --version")
11. run("uv venv .venv && source .venv/bin/activate && pip install -U pip")
12. run("source .venv/bin/activate && pip install -e \".[test,dev]\"")
13. run("source .venv/bin/activate && pip install pytest pytest-cov")
14. run("source .venv/bin/activate && python -m pytest --co -q")
15. run("source .venv/bin/activate && python -m pytest -x -ra -vv --tb=long 2>&1 | tee /tmp/test.log")
16. run("FAIL=$(grep -m1 '^FAILED ' /tmp/test.log | awk '{print $2}' | cut -d'::' -f1); source .venv/bin/activate && python -m pytest -x -ra -vv --tb=long \"\$FAIL\"")

Most likely root-cause hypothesis:
- The CI environment uses a different Python version or resolves to different (often newer) dependency versions than your local machine, exposing a latent incompatibility, deprecation, or behavioral change that only affects certain tests.

One tool call that would confirm it:
- run("docker run --rm -v $(pwd):/app -w /app python:3.11-slim bash -lc 'pip install -e \".[test,dev]\" && FAIL=$(python -m pytest -x -ra -vv --tb=long 2>&1 | grep -m1 \"^FAILED \" | awk \"{print \\$2}\" | cut -d\\\"::\\\" -f1) && python -m pytest -x -ra -vv --tb=long \"\\$FAIL\"'")
