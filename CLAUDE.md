# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **data archive**, not a codebase. It holds benchmark results for the
[localai-16gb-bench](https://github.com/ryanilano/localai-16gb-bench) project, which measures how
well local LLM + quant combinations fit and perform inside a single 16 GB GPU (RTX 4070 Ti SUPER).

There is **no build, no test suite, and no lint step** — nothing here is executable. Work in this
repo is about *reading, analyzing, and summarizing* the data, and keeping [FINDINGS.md](FINDINGS.md)
in sync with what the runs show.

## Where the data comes from

Results are **generated elsewhere** and pushed here by `scripts/publish-results.sh` in the
`localai-16gb-bench` harness repo. The harness commits arrive as `results: sync <run-id> from <host>`.
**Do not hand-author files under `results/`** — they are machine-produced provenance. The one file
meant to be edited by hand is `FINDINGS.md` (a living analysis summary); `README.md` documents the
archive layout.

If a fix concerns *how data is collected* (e.g. the mmproj-OOM quality-pass bug), the fix lands in the
**harness repo**, not here. This repo only records that it was diagnosed. See the FINDINGS.md
"Known failures" section for an example of that split.

## Layout

Each run is a self-contained, immutably-named folder `results/<date>_<time>-<host>/` (the run id is
sortable and unique, so runs never overwrite each other):

- `throughput.csv` — the machine-parseable source of truth. Columns:
  `label,quant,type,depth,pp_tok_s,tg_tok_s,vram_peak_mib,ram_used_peak_mib,status`
- `RUN.md` — human-readable report (results table + provenance), regenerated from the CSV + versions.
- `versions.txt` — provenance: driver, CUDA toolkit, llama.cpp build/commit, GPU/VRAM, power cap.
- `json/` — per-test `llama-bench` JSON plus raw logs, keyed by `<label>_d<depth>`.
- `quality/<label>/` — per-model answer `.md` files (only present on quality-pass runs).

`results/prefetch_*.log` are top-level page-cache warm-up logs, not part of any single run.

## Key concepts for reading the data

- A **fit sweep** = the same (model, quant) benchmarked at increasing context `depth` (0, 16384,
  32768, …). The interesting question is always *how deep can this config go before it OOMs*, so
  read a config as a curve across depths, not as a single row.
- `status` is `OK` or `FAIL`. **A `FAIL` with `vram_peak ≈ 2 MiB` never loaded** (missing/corrupt
  GGUF or unsupported quant) — this is categorically different from an OOM `FAIL` that loaded fine
  (~14–15 GiB) then ran out building the KV cache. Don't conflate them.
- **Distrust single-sample outliers**, especially depth-0 / first-measured rows (cold page-cache
  artifacts). Trust warm, repeated numbers. FINDINGS.md flags specific known-noisy rows.
- `type` is `dense` (27B) vs `moe` (35B). MoE offloads experts, so its VRAM stays low even at deep
  context — the two families behave nothing alike and should be compared with that in mind.

## Provenance is not uniform across runs

The earliest runs predate features and have gaps — always check what a run actually contains before
citing it. Documented exceptions live in [README.md](README.md) and the FINDINGS.md run index; e.g.
`2026-07-01_202112` has no `versions.txt` (CUDA version was footer-appended after the fact),
`2026-07-02_175923` lost its `json/` to the old flat layout, and a couple of runs have `json/` but no
`RUN.md`. When making claims about the hardware/software stack, cite the run's own `versions.txt`
rather than assuming the current stack applied to older runs.

## When updating FINDINGS.md

- It is the one narrative artifact; keep it faithful to the CSVs. Every quantitative claim should
  trace to a specific run's `throughput.csv` — cite the run id.
- Follow the global rule against fabricated stats: if a number isn't in a run's data, don't state it.
- **Throughput is validated; quality is not.** As of the latest runs, all quality passes recorded
  "no response" (mmproj OOM). Do not present quality/"which model is smarter" conclusions as settled
  until a clean quality pass lands.
