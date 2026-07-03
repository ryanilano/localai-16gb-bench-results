# 27B_IQ4_XS — 04_toolcall_plan

## Prompt

You are an autonomous coding agent with exactly these tools:
- read_file(path: string) -> string
- run(cmd: string) -> {stdout: string, exit_code: int}
- write_file(path: string, content: string) -> bool

Task: a Python project's test suite fails only in CI. Produce a step-by-step plan to reproduce and locate the failure locally, expressed as an ordered list of concrete tool calls with the exact arguments you would pass at each step (not prose). Assume nothing about the repo layout — discover it. After the plan, give the single most likely root-cause hypothesis for a "passes locally, fails in CI" test and the one tool call that would confirm it.

## Response

1. `run(cmd="find . -maxdepth 2 -type f \\( -name '*.yml' -o -name '*.yaml' -o -name 'tox.ini' -o -name 'pyproject.toml' -o -name 'pytest.ini' -o -name 'conftest.py' -o -name 'setup.cfg' \\) 2>/dev/null | head -10")`
2. `read_file(path=".github/workflows/ci.yml")`
3. `read_file(path="pyproject.toml")`
4. `read_file(path="conftest.py")`
5. `run(cmd="python -m pytest -v --tb=long 2>&1 | tee /tmp/local_run.log")`
6. `run(cmd="grep -E 'python-version|run:.*pytest|install.*requirements|env:' .github/workflows/ci.yml")`
7. `run(cmd="python -m venv .ci-env && source .ci-env/bin/activate && pip install --upgrade pip && pip install -e '.[test]' && python -m pytest -v --tb=long 2>&1 | tee /tmp/ci_repro.log")`
8. `run(cmd="python -c \"import locale, os; print(f'Locale: {locale.getlocale()}'); print(f'TZ: {os.environ.get(\\\"TZ\\\")}'); print(f'LC_ALL: {os.environ.get(\\\"LC_ALL\\\")}')\"")`
9. `run(cmd="python -m pytest -v --tb=long --no-cov -p no:randomly -p no:hypothesis 2>&1 | tee /tmp/isolated_run.log")`
10. `run(cmd="diff -u /tmp/local_run.log /tmp/ci_repro.log | head -50")`

**Most likely root-cause hypothesis:** CI environments default to the `C`/`POSIX` locale and `UTC` timezone, while local development machines typically use UTF-8 locales and local timezones. This causes tests that rely on implicit locale-dependent string formatting, sorting, or datetime parsing to fail in CI.

**Confirmation tool call:** `run(cmd="python -c \"import locale, os; print(f'Locale: {locale.getlocale()}'); print(f'TZ: {os.environ.get(\\\"TZ\\\")}'); print(f'LC_ALL: {os.environ.get(\\\"LC_ALL\\\")}')\"")`
