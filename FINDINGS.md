# Findings — localai-16gb-bench

_Working summary of benchmark results through 2026-07-03. This is a living notes file; the
authoritative data is each run's `throughput.csv` / `RUN.md` under `results/`._

## What we're testing

Goal: find the best local LLM + quant combos that **fit and stay useful inside a single 16 GB GPU**,
measured across increasing context depth (a "fit sweep").

- **Hardware:** NVIDIA RTX 4070 Ti SUPER, 16 GB (16376 MiB total, ~15943 MiB usable), power cap 250 W (285 W max)
- **Host:** `debian-llm`
- **Stack:** CUDA 13.3 (nvcc V13.3.73), driver 595.71.05, llama.cpp build **9859 (`4fc4ec554`)**, GCC 14.2.0
- **Metrics per (model, quant, depth):** `pp_tok_s` (prefill), `tg_tok_s` (token gen), `vram_peak_mib`, `ram_used_peak_mib`, `status`

Two model families under test:

| Family | Arch | Quants tried | Variants |
| --- | --- | --- | --- |
| 27B | dense (`qwen35 27B`, ~26.9 B params) | IQ4_XS, Q3_K_M, Q3_K_P, IQ3_M, (Q4_K_M ✗) | Heretic_Youssofal, HauhauCS_Balanced, NEO_CODE, **Heretic_NEO_CODE** |
| 35B | MoE (`35B`) | UD-IQ4_NL_XL, UD-Q4_K_XL, UD-Q3_K_M | Unsloth dynamic quants |

## Headline conclusions

1. **MoE 35B is the standout for the 16 GB budget.** With experts offloaded, VRAM stays at only
   **~2.8–6 GiB even out to 261 k context** — it never came close to the VRAM ceiling. Token-gen is
   the fastest of anything tested (**~47–58 tok/s** shallow, still ~33 tok/s at 261 k). Prefill is
   lower than dense (~600–860 tok/s). This is the pick when you want long context + fast generation.
   - `35B_UD-Q3_K_M` and `35B_UD-IQ4_NL_XL` are the best of the three; `UD-IQ4_NL_XL` gives the best
     prefill and highest effective precision among the MoE quants.

2. **Dense 27B is prefill-fast but VRAM-bound.** Prefill is much higher (~1400–1730 tok/s) but
   token-gen is lower (~30–41 tok/s) and VRAM fills fast:
   - **IQ4_XS** (best-quality dense that fits): ~15 GiB by 16 k depth, then **OOM/FAIL at 32 k**. Good
     quality but effectively capped near ~16 k context on this GPU.
   - **Q3_K_M / Q3_K_P**: ~13–14 GiB, reach 32 k–49 k, FAIL beyond.
   - **IQ3_M** (smallest dense): ~12.7 GiB at depth 0, sweeps all the way to **81 920 depth at
     ~15.4 GiB** — the best dense option for long context, at some quality cost.
   - **Q4_K_M dense always FAILs** — it needs ~15.3 GiB just to load and OOMs building KV. Q3 is the
     ceiling for dense quants at 4-bit-ish on 16 GB.

3. **Current focus: the Heretic NEO CODE merge (latest 2 runs).**
   - `27B_Heretic_NEO_CODE_IQ3_M` sweeps cleanly to **81 920 (80 k) context within 16 GB** (peak 15 432 MiB,
     tg ~28–41 tok/s) — see `2026-07-03_005125`.
   - `27B_Heretic_NEO_CODE_IQ4_XS` runs to 16 k but **FAILs at 32 k** (same IQ4_XS wall as the other
     dense variants) — see `2026-07-03_004208`.
   - Takeaway so far: **IQ3_M is the long-context config for this merge; IQ4_XS is the ~16 k / higher-quality config.**

## Rough decision guide

| Need | Pick |
| --- | --- |
| Max context (64 k+) + fastest generation | **35B MoE** — `UD-IQ4_NL_XL` or `UD-Q3_K_M` |
| Best dense quality, short/medium context (≤16 k) | **27B IQ4_XS** (incl. Heretic_NEO_CODE) |
| Dense + long context on 16 GB | **27B IQ3_M** (Heretic_NEO_CODE / NEO_CODE) |
| Fastest prefill | dense IQ4_XS (~1.7 k tok/s) |

## Known failures & data-quality caveats

- **Hard FAILs with `vram_peak = 2 MiB`** (`27B_..._Q3_K_L`, `35B_Heretic_HauhauCS` Q4_K_P): these
  never loaded — a `vram≈2` failure means missing/corrupt GGUF or an unsupported quant, **not** OOM.
  Distinct from the IQ4_XS depth-32 k OOMs (which load fine, then run out building KV at ~14–15 GiB).
- **Noisy first samples.** Some depth-0 / first-measured rows are cold-cache artifacts: e.g.
  `35B_UD-Q4_K_XL` d0 prefill swings 261→615 across runs, and `ram_used_peak` spikes (23 812, 15 157 MiB)
  appear only on the first sample after prefetch (page-cache), not on warm rows. Trust the warm, repeated
  numbers; treat single outliers with suspicion.
- **Quality passes still produce no answers — root cause now diagnosed (mmproj OOM).** Two quality
  runs exist and both recorded `(no response — see _server.log)` for every prompt:
  `2026-07-01_202112` (3 prompts) and the newer `2026-07-03_010929` (expanded to 9 prompts × 8 configs).
  The `2026-07-03_010929/_server.log` shows the cause: the `-hf` resolver auto-loads a **vision
  projector (mmproj/CLIP)** for these Qwen3.6 repos, and on the 16 GB card the ~888 MiB CLIP buffer
  `cudaMalloc failed: out of memory` _after_ the model is fully offloaded at `-ngl 99` — the server
  aborts on boot, so every prompt logs "no response". Throughput passed because the `llama-bench` path
  is text-only and never loads the projector.
  - **Fix applied in the harness** (`localai-16gb-bench`, not this results repo): `run-quality.sh`,
    `serve-27b-uncensored.sh`, and `serve-35b-moe.sh` now launch `llama-server` with `--no-mmproj`.
    Awaiting a re-run to confirm.
  - **Harness hardening (same change):** the server log is now per-model
    (`quality/<label>/_server.log`) instead of one clobbered `_server.log`, so a crash for one config
    no longer erases the next's log. `-ngl` is now per-config (CONFIGS field 6, default 99 = full
    offload) and a config that fails to boot auto-retries once with `-ngl -1` (llama.cpp auto-fit) —
    RUN.md records the effective ngl per model, so watch for `[ngl=-1 (auto-fit fallback)]`: that model
    ran with layers on CPU, not the full-offload regime the throughput numbers assume.
  - **All quality conclusions remain TODO; only throughput is validated so far.**
- **Provenance gaps in early runs** (documented in `README.md`): `2026-07-01_202112` predates version
  stamping (CUDA 13.1, footer-appended); `2026-07-02_175923` lost its `json/` to the old flat layout;
  `2026-07-02_221547` and `_223156` have `json/` but no `RUN.md`.

## Next steps

1. **Re-run the quality pass** now that the harness is fixed (`--no-mmproj` added to all `llama-server`
   launches). The `2026-07-03_010929` matrix (9 prompts × 8 configs) is ready to re-run; confirm it now
   produces real answers, then judge which merge is actually smarter — throughput tells us what fits and
   how fast, but not that. This remains the biggest open gap until a clean pass lands.
2. **Investigate the `vram=2` hard FAILs** (Q3_K_L dense, 35B Heretic HauhauCS Q4_K_P): confirm the GGUF
   exists/prefetched and the quant is supported by build 9859; re-run or drop from the matrix.
3. **Push the MoE fit sweep further.** 35B MoE never hit a VRAM wall (fine at 261 k). Extend to
   512 k / 1 M to find the *actual* max context on 16 GB, and record the tg curve as it degrades.
4. **Backfill `RUN.md`** for `2026-07-02_221547` and `_223156` from their existing `json/`.
5. **Confirm the IQ4_XS 32 k ceiling** is fundamental vs. tunable (flash-attn, KV cache quant e.g.
   `-ctk/-ctv q8_0`) — KV-cache quantization might push dense IQ4_XS past 16 k.
6. Optional: **power-cap sensitivity** (250 W vs 285 W) to see if the cap is limiting tg.

## Run index

| Run | Contents | Notes |
| --- | --- | --- |
| `2026-07-01_202112` | csv + json + quality | earliest; no versions.txt; quality empty |
| `2026-07-02_175923` | csv + RUN.md | bench-only; json lost |
| `2026-07-02_193446` | full | broad dense+MoE matrix |
| `2026-07-02_204121` | full | broad dense+MoE matrix |
| `2026-07-02_215109` | full | broad dense+MoE matrix |
| `2026-07-02_221547` | csv + json | no RUN.md |
| `2026-07-02_223156` | csv + json | deep MoE sweep (→131 k) |
| `2026-07-02_223927` | full | deep sweeps, MoE →261 k |
| `2026-07-03_004208` | full | **Heretic_NEO_CODE** IQ3_M + IQ4_XS |
| `2026-07-03_005125` | full | **Heretic_NEO_CODE** IQ3_M →80 k |
| `2026-07-03_010929` | quality | 9 prompts × 8 configs; all "no response" (mmproj OOM — harness now fixed) |
