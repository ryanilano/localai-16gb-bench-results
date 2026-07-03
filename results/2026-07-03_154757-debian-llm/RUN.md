# Agent-loop run — 2026-07-03_154757-debian-llm

**Scenario:** `fixbug`  ·  GEN=2048 QCTX=16384 TEMP=0.2

## Results

| config | verdict | detail |
| --- | --- | --- |
| 27B_IQ4_XS | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=648c med=2886ms |
| 35B_UD-Q4_K_XL | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=940c med=4774ms |
| 35B_UD-Q3_K_M | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=864c med=4025ms |
| 27B_HauhauCS_Balanced | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=619c med=2717ms |
| 27B_HauhauCS_Balanced_Q3_K_P | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=693c med=2702ms |
| 27B_NEO_CODE_IQ4_XS | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=590c med=2912ms |
| 27B_NEO_CODE_IQ3_M | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=559c med=2819ms |
| 27B_Heretic_NEO_CODE_IQ3_M | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=435c med=3279ms |
| 27B_Heretic_NEO_CODE_IQ4_XS | PASS | done=true tools=4/4 drift=0 turns=4 trunc=false think=449c med=2558ms |

_verdict PASS = scenario check passed. detail: tools=valid/total tool calls, drift=raw <tool_call> not structured, trunc=hit max_turns, think=reasoning chars burned, med=median request ms._

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-03 15:47:57 -0400
hostname:             debian-llm

## NVIDIA driver / GPU
gpu (driver,name,vram): 595.71.05, NVIDIA GeForce RTX 4070 Ti SUPER, 16376 MiB
power (limit,max):      250.00 W, 285.00 W
driver userspace:       NVRM version: NVIDIA UNIX x86_64 Kernel Module  595.71.05  Fri Apr 24 06:36:12 UTC 2026
GCC version:  gcc version 14.2.0 (Debian 14.2.0-19) 

## CUDA toolkit
nvcc release:           (nvcc unavailable)
/usr/local/cuda ->:     /usr/local/cuda-13.3

## llama.cpp
resolved binary:        /root/llama.cpp/build-13.3/bin/llama-server
source git commit:      4fc4ec5541b243957ae5099edb67372f8f3b550e
version output:
  version: 9859 (4fc4ec554)
  built with GNU 14.2.0 for Linux x86_64
```
