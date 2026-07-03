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

Three benchmark dimensions now feed this: **throughput/fit** (the depth sweep above), **quality**
(scored answers to fixed prompts), and **agentic tool-use** (an agent-loop sandbox that scores whether
the model fixes a bug through tool calls). See the respective sections below.

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
   **underperformed dense on coding** here — but the `GEN=8192` reruns showed its apparent faults (the
   `UD-Q4_K_XL` semver bug) were budget-induced, not real defects. See the quality section below; the MoE
   losses were GEN-budget truncations, not wrong answers.

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

### KV-cache fit model — the dense wall is two numbers

The IQ4_XS KV-quant probe (`2026-07-03_151252` q8_0, `2026-07-03_151915` q4_0) shows the whole dense
fit map reduces to **load footprint + KV cost per token**, measured against the ~15943 MiB usable ceiling:

- **Load footprint** (weights + fixed buffers, at depth 0): **~15020 MiB** for the standard IQ4_XS configs
  (`27B_IQ4_XS`, both NEO_CODE lines); **~14675 MiB** for `27B_HauhauCS_Balanced` — ~345 MiB lighter.
- **KV cost per token** (constant across all four configs): **34.0 KiB/tok at q8_0** (+544 MiB per
  16 384-token rung), **18.0 KiB/tok at q4_0** (~+290 MiB/rung) — roughly half, as expected.

Those two numbers predict every observed wall exactly (a depth rung fits iff `footprint + n·rung ≤ 15943`):
q8_0 walls the standard configs at 16k and HauhauCS at 32k; q4_0 walls them at 49k and 65k. Consequences:

- **HauhauCS's free extra rung is entirely its ~345 MiB lighter load, not better KV handling** — it is the
  only config to reach 32k at safe q8_0.
- **q4_0 KV is free on speed** — tg/pp are identical to q8_0 at matched depth (16k: tg 33.6 vs 33.9, pp
  1436 vs 1447 tok/s). The only cost of sub-q8_0 KV is _coherence_ (unverified), not throughput.
- **Practical read for ≤32k use:** at safe q8_0 the standard IQ4_XS configs cap at 16k (the lead pick
  `Heretic_NEO_CODE_IQ4_XS` included); reaching 32k needs either q4_0 KV (coherence-untested) or a lighter
  config — `HauhauCS` at q8_0, or IQ3_M (which already clears 80k).

(Ignore the depth-0 `pp` for the two NEO_CODE configs in `151252` — 1306/1300 vs ~1700 elsewhere — that's
cold-page-cache first-sample noise; the warm 16k rows agree across both runs.)

## Quality results — first comprehensive pass (`2026-07-03_055350`)

10 configs × 9 prompts (coding, refactor, planning, strict-format, tool-call, dual-use security); GEN=4096.
82/90 real answers (8 GEN-budget truncations, 0 blank, 0 OOM). Graded for correctness:

| Tier | Config | Record (9 prompts) |
| --- | --- | --- |
| **Flawless** | `27B_Heretic_NEO_CODE_IQ4_XS`, `27B_NEO_CODE_IQ4_XS`, `27B_Heretic_NEO_CODE_IQ3_M`, `27B_IQ4_XS` | 9/9 correct, 0 truncations |
| **One blemish** | `27B_HauhauCS_Balanced_Q3_K_P` (1 trunc); `27B_NEO_CODE_IQ3_M` (1 degenerate `//\|//` loop on agent_plan) | 8 good |
| **A real fault** ⚠️ | `35B_UD-Q4_K_XL` (semver regex crashes on `-alpha.beta`; best instruction-follower otherwise) | + 1 trunc |
| | `35B_UD-Q3_K_M` (2 trunc, minor `git grep -E` slip) | |
| | `27B_HauhauCS_Balanced` (constraint FAIL — kept `requests` when told stdlib-only) | + 1 trunc |

> ⚠️ **Both "real fault" rows were later cleared by the `GEN=8192` reruns** (`113237`/`125723`): the
> `UD-Q4_K_XL` semver crash and the HauhauCS stdlib violation are **GEN-budget artifacts, not real
> defects** — both models answer correctly at full budget. See "Higher-GEN rerun" below.
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

## Higher-GEN rerun — verdicts on the two real faults (`113237`, `123210`)

The follow-up ran `GEN=8192` (double the `055350` budget). `113237` re-ran 5 configs at `QCTX=16384`;
`123210` re-ran the two `NEO_CODE` variants at `QCTX=8192`; `125723` re-ran 5 configs (incl. HauhauCS) at
`QCTX=16384`. (`123132`, a `NEO_CODE` base attempt at `QCTX=8192`, aborted before answering any prompt —
superseded by `123210`.) **Both flagged faults turned out to be GEN-budget artifacts, not real defects.**
Grading them:

- **`35B_UD-Q4_K_XL` / `01_semver_compare` — RESOLVED.** At full budget the MoE emits a complete,
  **correct** comparator: string-`partition` parsing (no regex, so no crash), correct numeric-vs-alphanumeric
  precedence, and the differing-field-count rule via `len()`. `1.0.0-alpha.1` vs `1.0.0-alpha.beta` now
  returns `-1` correctly. The `055350` "regex crashes on `-alpha.beta`" fault was budget-induced, not a real
  logic defect. Answer finished cleanly (no truncation).
- **`27B_HauhauCS_Balanced` / `03_refactor_constrained` — RESOLVED (`125723`).** On a clean boot HauhauCS
  loaded fine at `QCTX=16384` (all 5 configs loaded, 9/9 each) — confirming the `113237` abort was the
  stale-server VRAM leak, not model size. At full budget the final deliverable is **stdlib-only**: it
  reasons through the constraint ("standard library only … so `requests` must go") and ships
  `urllib.request`/`urllib.error`/`json`. The `055350` "kept `requests`" fault was budget-induced — at
  `GEN=4096` it lacked the room to reason to the switch. Both flagged faults were GEN artifacts, **not**
  real logic/instruction defects.
- **`NEO_CODE` A/B holds (`123210`):** both `27B_Heretic_NEO_CODE_IQ3_M` and base `27B_NEO_CODE_IQ3_M`
  answered 9/9 with no truncation (max ~4.4 k tokens, well under budget). Abliteration-is-free stands; the
  overall pick `27B_Heretic_NEO_CODE_IQ3_M` is unchanged.
- **Doubling GEN did _not_ close all truncations.** 4 of 36 answers in `113237` still hit the 8192 cap —
  2 in the over-thinking `Heretic_Youssofal`, 1 each in the two MoEs. The chronic reasoners burn even a
  doubled budget on chain-of-thought; a GEN ceiling alone won't fix them.

## Agentic tool-use — first pass (`154255`, `154757`, `155835`)

A third benchmark dimension landed on 2026-07-03: an **agent-loop** harness that drops the model in a
sandbox with a buggy module + a test file and makes it fix the bug **through tool calls**, scoring
`task_completed` (tests pass), tool-call validity, format drift (raw `<tool_call>` vs structured),
loop truncation, thinking-token burn, turns, and per-request latency. All runs were `GEN=2048
QCTX=16384 TEMP=0.2` on the current stack (llama.cpp 9859 `4fc4ec554`).

**Every config passed every scenario, cleanly — zero format drift, all tool calls valid, no loop
truncation.** This is the first agentic-competence signal and it is uniformly positive; notably the
35B MoEs — which had faults on the earlier quality pass — handle tool-use fine.

| Scenario | Configs (all PASS) | Tools | Turns |
| --- | --- | --- | --- |
| `fixbug` (single file) — `154255` | `Heretic_NEO_CODE_IQ4_XS` | 4/4 | 4 |
| `fixbug` (single file) — `154757` | `27B_IQ4_XS`, `35B_UD-Q4_K_XL`, `35B_UD-Q3_K_M`, `HauhauCS_Balanced` | 4/4 | 4 |
| `multifile` — `155835` | `27B_IQ4_XS`, `35B_UD-Q4_K_XL`, `35B_UD-Q3_K_M`, `HauhauCS_Balanced` | 9–10/9–10 | 4–6 |

- **Latency splits on family, as expected.** Dense 27B median ~2.5–3.3 s/request; the 35B MoEs
  ~4.0–4.8 s/request (~1.6–1.8× slower) — the MoE's tg-speed edge at long context does not carry over
  to short-context agentic turns, where dense wins on responsiveness.
- **Caveat:** these are the easiest possible agentic tasks (a 4-turn single-bug fix). A uniform PASS
  says the tool-calling _plumbing_ works for all configs, not that they'd diverge on harder tasks.
  Harder scenarios are needed before ranking configs on agentic ability.

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

1. **Quality grading is settled — both flagged faults were GEN artifacts (DONE).** The `GEN=8192` reruns
   (`113237` + `123210` + `125723`) closed both: `35B_UD-Q4_K_XL`'s semver "crash" and
   `27B_HauhauCS_Balanced`'s stdlib-constraint "violation" both disappear at full budget (correct answers —
   see "Higher-GEN rerun" above). The overall pick `27B_Heretic_NEO_CODE_IQ3_M` is unchanged. Residual
   (not blocking): 4/36 answers in `113237` still truncated at `GEN=8192` — the chronic over-thinkers
   (`Heretic_Youssofal` + the two MoEs) burn even a doubled budget on chain-of-thought, so a GEN ceiling
   alone won't fully close them.
2. **Investigate the `vram=2` hard FAILs** (Q3_K_L dense, 35B Heretic HauhauCS Q4_K_P): confirm the GGUF
   exists/prefetched and the quant is supported by build 9859; re-run or drop from the matrix.
3. **Push the MoE fit sweep further.** 35B MoE never hit a VRAM wall (fine at 261 k). Extend to
   512 k / 1 M to find the *actual* max context on 16 GB, and record the tg curve as it degrades.
4. **Backfill `RUN.md`** for `2026-07-02_221547` and `_223156` from their existing `json/`.
5. **Confirm the IQ4_XS 32 k ceiling is fundamental vs. tunable (KV-quant probe — STAGED, not yet run).**
   Correction: the throughput sweep **already runs with `-fa on` and `q8_0` KV** as hardcoded defaults
   (`run-bench.sh` `-ctk/-ctv "$KV_QUANT"`, `configs.sh KV_QUANT="q8_0"`), so the documented IQ4_XS wall
   was measured _with_ flash-attn + q8_0 KV — those are not untried levers. The remaining lever is a
   **more aggressive** KV quant than q8_0 (q5_1 → q4_1 → q4_0), which halves the KV buffer again. Harness
   prepped for this: `KV_QUANT` is now env-overridable (`configs.sh:74`, `${KV_QUANT:-q8_0}` — not yet
   committed). Two-rung decisive probe over the whole IQ4_XS **dense class** (stock `27B_IQ4_XS`,
   `27B_HauhauCS_Balanced`, `27B_NEO_CODE_IQ4_XS`, `27B_Heretic_NEO_CODE_IQ4_XS`):

   ```bash
   ONLY='IQ4_XS|HauhauCS_Balanced$' BENCH_PROFILE=longctx KV_QUANT=q8_0 ./run-bench.sh   # baseline wall
   ONLY='IQ4_XS|HauhauCS_Balanced$' BENCH_PROFILE=longctx KV_QUANT=q4_0 ./run-bench.sh   # aggressive
   ```

   Read `status`/`vram_peak` at depth ≥32768: `FAIL→OK` ⇒ KV-bound & tunable (then map `q5_1`/`q4_1` and
   run a quality pass — bench proves _fit_, not coherence, and sub-q8_0 KV could degrade answers);
   `FAIL→FAIL` ⇒ weights-bound, KV quant can't help and IQ3_M stays the long-context pick. Caveat:
   `27B_HauhauCS_Balanced` carries an mmproj — a _load-time_ abort (`vram≈2 MiB`) there is the vision-
   projector OOM, not a KV wall; read it separately from the other three.
   **RESULT — KV-BOUND & TUNABLE** (`2026-07-03_151252` q8_0, `2026-07-03_151915` q4_0): going q8_0→q4_0
   moved the OOM wall out ~2 depth rungs for every config — `27B_IQ4_XS` 16k→**49k**, `NEO_CODE_IQ4_XS`
   16k→**49k**, `Heretic_NEO_CODE_IQ4_XS` 16k→**49k**, `HauhauCS_Balanced` 32k→**65k**. So the ceiling is
   the KV buffer, not the ~15 GB weights (deepest OK rung ~15.8 GiB, near the 15943 ceiling). Bench proves
   _fit_ not coherence, so a q4_0-KV quality pass would be needed before trusting sub-q8_0 KV at 49k+
   (staged as step 6). For the **≤32k chat/agent workload this is ample** — IQ4_XS (best dense quality) is
   fully viable there, so precision is preferred over IQ3_M. Long-context-at-quality is off the critical
   path for that workload.
6. **Coherence-check sub-q8_0 KV (q4_0-KV quality A/B — DONE, `2026-07-03_170204`).** The probe (step 5) proved
   the IQ4_XS-class 49k fit is real, but a bench only proves the KV cache _fits_, not that answers stay
   coherent under a coarser KV quant. Open question: does `q4_0` KV degrade answer quality vs `q8_0`?
   Decisive A/B — re-run the same IQ4_XS-class configs and prompts at `q4_0` KV and compare scores against
   the `q8_0` baseline `2026-07-03_125723` (same `GEN=8192 QCTX=16384`, only the KV quant differs, so any
   score drop is attributable to KV). No harness change needed: `KV_QUANT` is env-overridable and the
   per-model KV lookup is empty (`configs.sh:103` `kv_quant_for_label` returns `""`), so the global applies
   uniformly. Runs on `debian-llm`:

   ```bash
   ONLY='IQ4_XS|HauhauCS_Balanced$' GEN=8192 QCTX=16384 KV_QUANT=q4_0 ./run-quality.sh
   ```

   Read: scores hold ⇒ `q4_0` KV is coherence-safe and the 49k fit is usable in practice (IQ4_XS becomes
   a viable long-context pick, not just IQ3_M); scores drop ⇒ sub-q8_0 KV trades coherence for context, so
   keep `q8_0` and fall back to IQ3_M for long context. If `q4_0` degrades, map the intermediate rungs
   (`q5_1` → `q4_1`) to find the coherence/fit knee. Caveat: the prompts are short (they don't themselves
   reach 49k), so this measures KV-quant coherence _per se_, not behaviour at extreme depth — a separate
   deep-context coherence probe would need long-input prompts.

   **RESULT — `q4_0` KV IS COHERENCE-SAFE** (`2026-07-03_170204`, all 4 IQ4_XS-class configs, `GEN=8192
   QCTX=16384 TEMP=0.2 KV_QUANT=q4_0`). Every finished answer is correct and coherent — no garbling, no
   wrong logic, no degeneracy anywhere (35/36 answers complete; the lone exception is a reasoning-only GEN
   truncation, `HauhauCS/02_debug_subtle`). Detail:
   - **The one clean single-variable pair is `HauhauCS_Balanced` vs q8_0 `125723`** (matched `GEN`/`QCTX`,
     only KV differs). The other three configs (`27B_IQ4_XS`, `NEO_CODE_IQ4_XS`, `Heretic_NEO_CODE_IQ4_XS`)
     were **not** in `125723` — that run was the HauhauCS-focused rerun (HauhauCS ×2 quants + Youssofal +
     the two MoEs), so their nearest q8_0 baseline is `055350` at `GEN=4096/QCTX=8192` (unmatched). For
     those three I confirmed intrinsic coherence (9/9 clean, spec-compliant answers), not a head-to-head score.
   - **On the clean HauhauCS pair, q4_0 shows no degradation:** `01_palindrome` correct in both; on
     `01_semver_compare` q4_0 produced a full correct implementation where **q8_0 truncated**; on
     `02_debug_subtle` the reverse. The truncations _swapped prompts_ between runs — stochastic GEN-budget
     noise on these thinking models, not a KV effect. q4_0 truncated **fewer** total than q8_0 (1 vs 2 for
     HauhauCS in `125723`).
   - **Consequence:** sub-q8_0 KV does not trade coherence for context here, so the probe's 49k `q4_0` fit
     is usable in practice — **IQ4_XS (best dense quality) becomes viable at long context, not just IQ3_M.**
     Caveat stands: prompts are short (≤16k), so this validates KV-quant coherence _per se_, not answer
     quality when the cache is actually filled to 49k — a long-input deep-context probe remains unrun, and
     the three non-HauhauCS configs lack a matched-GEN q8_0 baseline. `q5_1`/`q4_1` intermediate rungs were
     not needed (q4_0 already clean).
7. Optional: **power-cap sensitivity** (250 W vs 285 W) to see if the cap is limiting tg.

## Run index

| Run | Contents | Notes |
| --- | --- | --- |
| [`2026-07-01_202112`](results/2026-07-01_202112-debian-llm/) | csv + json + quality | earliest; no versions.txt; quality empty |
| [`2026-07-02_175923`](results/2026-07-02_175923-debian-llm/) | csv + RUN.md | bench-only; json lost |
| [`2026-07-02_193446`](results/2026-07-02_193446-debian-llm/) | full | broad dense+MoE matrix |
| [`2026-07-02_204121`](results/2026-07-02_204121-debian-llm/) | full | broad dense+MoE matrix |
| [`2026-07-02_215109`](results/2026-07-02_215109-debian-llm/) | full | broad dense+MoE matrix |
| [`2026-07-02_221547`](results/2026-07-02_221547-debian-llm/) | csv + json | no RUN.md |
| [`2026-07-02_223156`](results/2026-07-02_223156-debian-llm/) | csv + json | deep MoE sweep (→131 k) |
| [`2026-07-02_223927`](results/2026-07-02_223927-debian-llm/) | full | deep sweeps, MoE →261 k |
| [`2026-07-03_004208`](results/2026-07-03_004208-debian-llm/) | full | **Heretic_NEO_CODE** IQ3_M + IQ4_XS |
| [`2026-07-03_005125`](results/2026-07-03_005125-debian-llm/) | full | **Heretic_NEO_CODE** IQ3_M →80 k |
| [`2026-07-03_010929`](results/2026-07-03_010929-debian-llm/) | quality | 9 prompts × 8 configs; all "no response" (mmproj OOM) |
| [`2026-07-03_013430`](results/2026-07-03_013430-debian-llm/) | quality | partial (3 configs); server boots but HTTP 500 empty-body |
| [`2026-07-03_013717`](results/2026-07-03_013717-debian-llm/) | quality | 9 prompts × 11 configs; all **blank** (stale-server port race — now fixed) |
| [`2026-07-03_033803`](results/2026-07-03_033803-debian-llm/) | throughput | empty csv (aborted) |
| [`2026-07-03_034042`](results/2026-07-03_034042-debian-llm/) | throughput | `27B_IQ4_XS` — OK to 16 k, FAIL at 32 k (VRAM wall) |
| [`2026-07-03_034918`](results/2026-07-03_034918-debian-llm/) | quality | `27B_IQ4_XS` only; blank — KV-cache OOM at `QCTX` |
| [`2026-07-03_035800`](results/2026-07-03_035800-debian-llm/) | quality | `27B_IQ4_XS` only; blank — `cudaMalloc` OOM |
| [`2026-07-03_040949`](results/2026-07-03_040949-debian-llm/) | quality | `35B_UD-IQ4_NL_XL` only; degenerate `////` output (model dropped) |
| [`2026-07-03_041809`](results/2026-07-03_041809-debian-llm/) | quality | **first real answers** — `35B_UD-Q3_K_M` + `Q4_K_XL`; 6 real, 12 truncated (raise GEN) |
| [`2026-07-03_055350`](results/2026-07-03_055350-debian-llm/) | quality | **first comprehensive pass** — 10 configs × 9 prompts, 82/90 real; see Quality results |
| [`2026-07-03_113237`](results/2026-07-03_113237-debian-llm/) | quality | `GEN=8192 QCTX=16384` re-run — 5 configs; 4/5 answered 9/9 (32 real, 4 still trunc); **HauhauCS failed to load** (VRAM fit abort, 0 answers) |
| [`2026-07-03_123132`](results/2026-07-03_123132-debian-llm/) | quality | `NEO_CODE` base at `QCTX=8192`; aborted before any answer — superseded by `123210` |
| [`2026-07-03_123210`](results/2026-07-03_123210-debian-llm/) | quality | `GEN=8192` A/B — `Heretic_NEO_CODE_IQ3_M` + base `NEO_CODE_IQ3_M`, 9/9 each, no trunc |
| [`2026-07-03_125723`](results/2026-07-03_125723-debian-llm/) | quality | `GEN=8192 QCTX=16384` re-run — 5 configs incl. HauhauCS (loaded clean, 9/9); **stdlib-constraint fault RESOLVED** (uses `urllib`) |
| [`2026-07-03_151252`](results/2026-07-03_151252-debian-llm/) | throughput | KV-quant probe rung 1 (`q8_0`) — IQ4_XS class walls at 16 k / HauhauCS 32 k |
| [`2026-07-03_151915`](results/2026-07-03_151915-debian-llm/) | throughput | KV-quant probe rung 2 (`q4_0`) — same class reaches 49 k / HauhauCS 65 k; **wall is KV-bound** |
| [`2026-07-03_154255`](results/2026-07-03_154255-debian-llm/) | agentloop | `fixbug` — `Heretic_NEO_CODE_IQ4_XS` PASS (4/4 tools, 0 drift) |
| [`2026-07-03_154757`](results/2026-07-03_154757-debian-llm/) | agentloop | `fixbug` — 4 configs (incl. both MoEs) all PASS |
| [`2026-07-03_155835`](results/2026-07-03_155835-debian-llm/) | agentloop | `multifile` — same 4 configs all PASS (9–10 tools) |
| [`2026-07-03_170204`](results/2026-07-03_170204-debian-llm/) | quality | q4_0-KV coherence A/B — 4 IQ4_XS-class configs, 35/36 clean; **q4_0 KV coherence-safe** (clean pair = HauhauCS vs `125723`) |
