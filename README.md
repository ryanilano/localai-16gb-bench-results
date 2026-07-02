# localai-16gb-bench-results

Private result archive for [localai-16gb-bench](https://github.com/ryanilano/localai-16gb-bench).

Results publish here via `scripts/publish-results.sh` in that repo. **Each run is its own
self-contained folder** under `results/`, named `<date>_<time>-<host>`:

```
results/
  2026-07-02_175923-debian-llm/
    throughput.csv     # throughput + fit sweep (machine-parseable)
    versions.txt       # provenance: driver, CUDA toolkit, llama.cpp build, GPU/VRAM, power cap
    RUN.md             # human-readable report: results table + provenance
    json/              # per-test llama-bench JSON + logs
    quality/<label>/   # per-model answer .md files (quality passes only)
```

The run id (`<date>_<time>-<host>`) is sortable and unique, so runs never overwrite each other.

## Notes on the earliest runs
- `2026-07-01_202112-debian-llm/` predates the version-stamping feature: it has `throughput.csv`
  (with a retroactively-appended, operator-confirmed provenance footer — CUDA 13.1), `json/`, and
  `quality/`, but no `versions.txt`/`RUN.md`.
- `2026-07-02_175923-debian-llm/` was a bench-only pass; its `json/` was lost to the old flat-layout
  overwrite before the per-run-folder fix landed, so only `throughput.csv` + `versions.txt` + `RUN.md`
  remain.
