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
| Max context (64 k+) + fastest generation | **35B MoE** — `UD-Q3_K_M` (`UD-IQ4_NL_XL` dropped: degenerate output) |
| Best dense quality, short/medium context (≤16 k) | **27B IQ4_XS** (incl. Heretic_NEO_CODE) |
| Dense + long context on 16 GB | **27B IQ3_M** (Heretic_NEO_CODE / NEO_CODE) |
| Fastest prefill | dense IQ4_XS (~1.7 k tok/s) |

### 35B MoE quant pick: Q3_K_M vs Q4_K_XL

**`UD-Q3_K_M` is the better default.** Both are the same MoE with experts offloaded, so both fit trivially
(VRAM ~2.8→6 GiB out to 261 k) — the difference is speed, RAM, and precision:

| | Q3_K_M | Q4_K_XL |
| --- | --- | --- |
| Prefill (pp tok/s) | **471–848** | 249–635 |
| Token-gen (tg tok/s) | 48–57 → 33 @261 k | 48–54 → 27 @261 k |
| Model size (system RAM) | **15.45 GiB** | 20.81 GiB |
| Precision | 3-bit-ish | **4-bit-ish** |

Q3_K_M has clearly higher prefill (esp. deep context), matches tg shallow and beats it deep, and needs
~5.4 GiB less RAM. On quality it was **indistinguishable** from Q4_K_XL in `2026-07-03_041809` — but that
rests on only **2 prompts where both models emitted a final answer** (the rest truncated on GEN), so it is
not a certified tie. Q4_K_XL's higher precision _should_ give it a marginal reasoning edge; reserve judgment
until a GEN-raised full pass can actually measure it. **For now: Q3_K_M — faster, lighter, no measured quality cost.**

## Known failures & data-quality caveats

- **Hard FAILs with `vram_peak = 2 MiB`** (`27B_..._Q3_K_L`, `35B_Heretic_HauhauCS` Q4_K_P): these
  never loaded — a `vram≈2` failure means missing/corrupt GGUF or an unsupported quant, **not** OOM.
  Distinct from the IQ4_XS depth-32 k OOMs (which load fine, then run out building KV at ~14–15 GiB).
- **Noisy first samples.** Some depth-0 / first-measured rows are cold-cache artifacts: e.g.
  `35B_UD-Q4_K_XL` d0 prefill swings 261→615 across runs, and `ram_used_peak` spikes (23 812, 15 157 MiB)
  appear only on the first sample after prefetch (page-cache), not on warm rows. Trust the warm, repeated
  numbers; treat single outliers with suspicion.
- **Quality pipeline: fixed after a three-failure chain — first real answers landed `2026-07-03_041809`.**
  Getting any gradable answer took clearing three distinct harness failures, each exposed only after the
  previous was fixed. The historical chain:
  1. **mmproj OOM (`2026-07-01_202112`, `2026-07-03_010929`).** Both recorded `(no response — see
     _server.log)` for every prompt. Cause: the `-hf` resolver auto-loads a **vision projector
     (mmproj/CLIP)** for these Qwen3.6 repos, and on the 16 GB card the ~888 MiB CLIP buffer
     `cudaMalloc failed: out of memory` _after_ the model is fully offloaded at `-ngl 99` — the server
     aborts on boot. **Fixed** in the harness (`--no-mmproj` on all `llama-server` launches).
  2. **Empty request bodies (`2026-07-03_013430`).** With mmproj fixed the server now *boots*, but every
     request returned HTTP 500 `parse_error … attempting to parse an empty input` — the harness sent
     empty JSON. Incomplete run: only 3 configs produced files, `35B_UD-Q3_K_M` produced 0.
  3. **Stale-server port race (`2026-07-03_013717`).** The most complete matrix (11 configs × 9 prompts)
     and the closest to a real pass — but all 99 answers are **blank**. Every `_server.log` shows
     `couldn't bind HTTP server socket … port 8080 … exiting`: a **stale `llama-server` already held
     :8080** (a leftover — the quality pass and both `serve-*.sh` scripts all default to 8080). Each
     freshly launched server died on bind, but the stale server answered the harness's `/health` poll,
     so `start_server` mistook it for ours and sent every prompt to the wrong (or empty) server;
     `stop_server` then killed only the just-died PID, so the stale server survived all 11 configs.
     (The answers are truly *blank* rather than the `(no response)` marker because the stale server
     returned `content: ""`, and jq's `//` fallback only fires on `null`, not empty string.)
  - **Fix applied in the harness** (`localai-16gb-bench`, branch `fix/quality-port-lifecycle`): after
    `pkill`-ing stale servers, poll until `:8080` is actually free before binding; and in the readiness
    loop, confirm our own `SRV_PID` is alive **before** trusting a `/health` 200, so a foreign server
    can never masquerade as ready (a bind failure now yields a clean skip, not silent blanks). The
    empty-content case is also now guarded so a blank can't pose as an answer. **Awaiting a re-run to
    confirm.** Note the maintainer had already independently added the `pkill` and empty/`reasoning_content`
    handling; this fix closes the remaining health-check ordering gap.
  - **Harness hardening (earlier change):** the server log is now per-model
    (`quality/<label>/_server.log`) instead of one clobbered `_server.log`, so a crash for one config
    no longer erases the next's log. `-ngl` is per-config (CONFIGS field 6, default 99 = full offload)
    and a config that fails to boot auto-retries once with `-ngl -1` (llama.cpp auto-fit) — RUN.md
    records the effective ngl per model, so watch for `[ngl=-1 (auto-fit fallback)]`: that model ran
    with layers on CPU, not the full-offload regime the throughput numbers assume.
  - **Pipeline now works end-to-end (runs `033803`–`041809`, live operator debugging on `debian-llm`).**
    Server boots cleanly, no mmproj/port/blank failures. Real answers appear from `2026-07-03_041809`:
    `35B_UD-Q3_K_M` and `35B_UD-Q4_K_XL` produced correct output where they finished — e.g. Q4_K_XL's
    palindrome is a clean, correct `unittest` solution and Q3_K_M's strict-format is a **flawless** JSON
    array (exact keys/values, tags sorted ascending, no prose, one line). First real quality signal, and
    it's promising — but only a handful of answers so far; no ranking is justified yet.
  - **Two model-side blockers remain (not harness bugs):**
    1. **Reasoning-budget truncation.** These Qwen3.6 MoE models are _thinking_ models; `GEN=768`
       max_tokens is too small, so the think phase eats the budget and 12 of 18 answers in `041809`
       are `⚠️ reasoning only — no final answer (raise GEN)`. Fix: raise `GEN` substantially (or cap the
       think phase) before a full pass.
    2. **`35B_UD-IQ4_NL_XL` emits degenerate output** (endless `////`, reasoning-only) — see `040949`.
       Already diagnosed and **dropped from `CONFIGS`** by the maintainer (`5763692`).
  - **Dense `27B_IQ4_XS` can't hold the quality-pass context on 16 GB** (`034918`, `035800`): the server
    boots but OOMs allocating the KV cache (`failed to allocate buffer for kv cache` / `cudaMalloc failed`),
    consistent with the throughput finding that IQ4_XS is VRAM-bound. Lower `QCTX` or use a smaller quant
    (IQ3_M) for its quality pass.
  - **Only throughput is fully validated; quality is now _unblocked_ but not yet complete.** Once `GEN` is
    raised, re-run the full matrix for a real cross-model quality comparison.
- **Provenance gaps in early runs** (documented in `README.md`): `2026-07-01_202112` predates version
  stamping (CUDA 13.1, footer-appended); `2026-07-02_175923` lost its `json/` to the old flat layout;
  `2026-07-02_221547` and `_223156` have `json/` but no `RUN.md`.

## Next steps

1. **Run the full quality matrix with enough answer budget.** The pipeline works (`2026-07-03_041809`
   produced real, correct answers), but the answer limit (`GEN`) truncates these reasoning models
   mid-think: `GEN=768` left answers blank, and even `GEN=2048` truncated 12 of 18 answers in `041809`
   (that run used 2048 — the think phase alone exceeds it on debug/refactor/multi-step prompts). The
   harness default is now `GEN=4096` (fits short prompts + most reasoning within `QCTX=8192`). **For a
   deep MoE quality pass, drive it wider:**

   ```bash
   GEN=8192 QCTX=16384 ./run-quality.sh      # MoE only — big think budget + room in the window
   ```

   Keep `QCTX` modest for the **VRAM-bound dense quants** (`27B_IQ4_XS` OOMs on KV even at 8192 — use a
   lower `QCTX` or the `IQ3_M` quant for its pass). Also drop the degenerate `35B_UD-IQ4_NL_XL`. Then run
   all configs × prompts and judge which merge is actually smarter. Biggest open gap until a clean pass lands.
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
| `2026-07-03_010929` | quality | 9 prompts × 8 configs; all "no response" (mmproj OOM) |
| `2026-07-03_013430` | quality | partial (3 configs); server boots but HTTP 500 empty-body |
| `2026-07-03_013717` | quality | 9 prompts × 11 configs; all **blank** (stale-server port race — now fixed) |
| `2026-07-03_033803` | throughput | empty csv (aborted) |
| `2026-07-03_034042` | throughput | `27B_IQ4_XS` — OK to 16 k, FAIL at 32 k (VRAM wall) |
| `2026-07-03_034918` | quality | `27B_IQ4_XS` only; blank — KV-cache OOM at `QCTX` |
| `2026-07-03_035800` | quality | `27B_IQ4_XS` only; blank — `cudaMalloc` OOM |
| `2026-07-03_040949` | quality | `35B_UD-IQ4_NL_XL` only; degenerate `////` output (model dropped) |
| `2026-07-03_041809` | quality | **first real answers** — `35B_UD-Q3_K_M` + `Q4_K_XL`; 6 real, 12 truncated (raise GEN) |
