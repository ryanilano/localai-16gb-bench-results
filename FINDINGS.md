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
   - On throughput, `35B_UD-Q3_K_M` and `35B_UD-IQ4_NL_XL` are the best of the three. **But
     `UD-IQ4_NL_XL` was later dropped — it emits degenerate output on the quality pass** (see below), so
     `UD-Q3_K_M` is the usable MoE pick. And on the coding quality pass the MoE lost to dense NEO_CODE
     (conclusion 4) — MoE's edge is long-context + tg speed, not coding correctness.

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

4. **On quality (first comprehensive pass, `2026-07-03_055350`), the dense NEO_CODE line wins — and
   abliteration is free.** The Heretic-abliterated NEO_CODE variants scored **identically** to the base on
   9 coding/reasoning/format prompts, and IQ3_M matched IQ4_XS — so **`27B_Heretic_NEO_CODE_IQ3_M` is the
   pick: top-tier quality _and_ 80 k context.** The 35B MoE, the throughput champion, actually
   **underperformed dense on coding** here (a real semver bug in `UD-Q4_K_XL`, plus truncations). See the
   quality section below; note some MoE losses are GEN-budget truncations, not wrong answers.

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

### Dense quality-pass fit (all dense quants run at the default QCTX=8192 / q8_0)

**Every active dense quant boots and answers cleanly at the default `QCTX=8192` / `q8_0` KV** — no
per-model caps needed. This corrects an earlier misdiagnosis: `27B_IQ4_XS` OOM'd allocating its 2720 MiB
KV buffer in `2026-07-03_034918`/`_035800`, which looked like the ~15 GB weights leaving no room for the
KV window. But both those runs **predate the stale-server port-race fix** (`b3a45fe`), and the real cause
was a killed-wrong-PID server still **holding VRAM**. Post-fix, `2026-07-03_055350` ran all 8 dense configs
(IQ4_XS included) with **0 OOM**, and the throughput sweep independently shows IQ4_XS's full 16 k footprint
is only ~15.5 GiB (< 15943). The per-model `qctx_for_label` / `kv_quant_for_label` lookups in `configs.sh`
are therefore **empty** (commit `0e1c60d`) — the machinery stays for any genuine future OOM, but no quant
currently needs it.

## Quality results — first comprehensive pass (`2026-07-03_055350`)

10 configs × 9 prompts (coding, refactor, planning, strict-format, tool-call, dual-use security); GEN=4096.
82/90 real answers (8 GEN-budget truncations, 0 blank, 0 OOM). Graded for correctness:

| Tier | Config | Record (9 prompts) |
| --- | --- | --- |
| **Flawless** | `27B_Heretic_NEO_CODE_IQ4_XS`, `27B_NEO_CODE_IQ4_XS`, `27B_Heretic_NEO_CODE_IQ3_M`, `27B_IQ4_XS` | 9/9 correct, 0 truncations |
| **One blemish** | `27B_HauhauCS_Balanced_Q3_K_P` (1 trunc); `27B_NEO_CODE_IQ3_M` (1 degenerate `//\|//` loop on agent_plan) | 8 good |
| **A real fault** | `35B_UD-Q4_K_XL` (semver regex crashes on `-alpha.beta`; best instruction-follower otherwise) | + 1 trunc |
| | `35B_UD-Q3_K_M` (2 trunc, minor `git grep -E` slip) | |
| | `27B_HauhauCS_Balanced` (constraint FAIL — kept `requests` when told stdlib-only) | + 1 trunc |
| **Worst** | `27B_Heretic_Youssofal_Q3_K_M` (4/9 truncated + bailed on tool-call — chronic over-thinking) | |

Conclusions:

- **Abliteration is free (NEO_CODE line):** Heretic-abliterated == base on coding quality — answers the
  open question in `configs.sh`. Both IQ3_M and IQ4_XS variants are top-tier.
- **IQ3_M ≈ IQ4_XS on quality**, so IQ3_M wins overall (same quality, lighter, 80 k context). **Overall
  pick: `27B_Heretic_NEO_CODE_IQ3_M`.**
- **MoE underperformed dense on this coding pass** — a genuine `UD-Q4_K_XL` semver crash plus truncations.
  Caveat: most MoE losses are GEN=4096 truncations (they think longer), not wrong answers; a higher-GEN
  rerun would give them a fairer shot.
- **Two non-discriminating prompts:** `05_strict_format` — all 10 produced identical perfect JSON;
  `06_security_dualuse` — all 10 (censored _and_ abliterated) complied with sound defensive SQLi answers,
  so it does **not** test abliteration. A prompt a standard model actually refuses is needed for that.
- Verdicts are from LLM graders; the **FAIL claims** (Q4_K_XL semver, HauhauCS constraint, the two
  degenerate-loop outputs) are the ones worth an eyeball since they set the ranking.

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
    empty-content case is also now guarded so a blank can't pose as an answer. **Confirmed fixed** —
    `2026-07-03_055350` booted all 10 configs with 0 blank / 0 port-bind. Note the maintainer had already
    independently added the `pkill` and empty/`reasoning_content` handling; this closed the health-check gap.
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
  - **The `27B_IQ4_XS` KV-alloc OOMs (`034918`, `035800`) were the stale-server VRAM leak, not a real
    IQ4_XS limit** — both predate the port-race fix. Post-fix (`055350`) IQ4_XS runs clean at default
    `QCTX=8192`. See the "Dense quality-pass fit" note above; the per-model KV caps were removed (`0e1c60d`).
  - **Comprehensive quality pass landed (`2026-07-03_055350`)** — 82/90 real answers, all 10 configs.
    See the "Quality results" section above. Only remaining gap: 8 GEN=4096 truncations (mostly the MoE
    and the over-thinking Youssofal merge); a higher-GEN rerun would close them.
- **Provenance gaps in early runs** (documented in `README.md`): `2026-07-01_202112` predates version
  stamping (CUDA 13.1, footer-appended); `2026-07-02_175923` lost its `json/` to the old flat layout;
  `2026-07-02_221547` and `_223156` have `json/` but no `RUN.md`.

## Next steps

1. **Re-run the quality matrix at higher GEN to close the 8 truncations.** The comprehensive pass
   (`2026-07-03_055350`, GEN=4096) truncated 8 of 90 answers — mostly the two MoE configs and the
   over-thinking `Heretic_Youssofal` merge, which run out of budget mid-think rather than answering wrong.
   Re-run with a bigger answer budget so the ranking rests on finished answers (and the MoE gets a fair
   shot at the coding prompts it currently loses to truncation):

   ```bash
   GEN=8192 QCTX=16384 ./run-quality.sh      # all configs now fit this; KV caps removed (0e1c60d)
   ```

   Then confirm whether `35B_UD-Q4_K_XL`'s semver crash and `27B_HauhauCS_Balanced`'s stdlib-constraint
   violation persist with a full budget (those are real faults, not truncations).
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
| `2026-07-03_055350` | quality | **first comprehensive pass** — 10 configs × 9 prompts, 82/90 real; see Quality results |
| `2026-07-03_113237` | quality | partial re-run — 5 configs (HauhauCS, Youssofal, IQ4_XS + 2 MoE), 32 real |
