# localai-16gb-bench-results

Private result archive for [localai-16gb-bench](https://github.com/ryanilano/localai-16gb-bench).

Each run publishes here via `scripts/publish-results.sh` in that repo:

- `throughput_<run>.csv` — machine-parseable throughput + fit sweep
- `versions_<run>.txt` — provenance stamp (driver, CUDA toolkit, llama.cpp build, GPU/VRAM, power cap)
- `RUN_<run>.md` — human-readable report: results table + provenance
- `quality/` — per-model answer `.md` files and their provenance

Run id (`<run>`) is `YYYY-MM-DD_HHMMSS-<host>`, so results are sortable and never overwrite each other.
