# Agent-loop run — 2026-07-03_155835-debian-llm

**Scenario:** `multifile`  ·  GEN=2048 QCTX=16384 TEMP=0.2

## Results

| config | verdict | detail |
| --- | --- | --- |
| 27B_IQ4_XS | PASS | done=true tools=9/9 drift=0 turns=5 trunc=false think=760c med=2596ms |
| 35B_UD-Q4_K_XL | PASS | done=true tools=9/9 drift=0 turns=5 trunc=false think=642c med=4219ms |
| 35B_UD-Q3_K_M | PASS | done=true tools=9/9 drift=0 turns=4 trunc=false think=760c med=4498ms |
| 27B_HauhauCS_Balanced | PASS | done=true tools=10/10 drift=0 turns=6 trunc=false think=829c med=3332ms |
| 27B_HauhauCS_Balanced_Q3_K_P | PASS | done=true tools=10/10 drift=0 turns=6 trunc=false think=764c med=2567ms |
| 27B_NEO_CODE_IQ4_XS | PASS | done=true tools=10/10 drift=0 turns=6 trunc=false think=697c med=2356ms |
| 27B_NEO_CODE_IQ3_M | PASS | done=true tools=9/9 drift=0 turns=5 trunc=false think=888c med=2548ms |
| 27B_Heretic_NEO_CODE_IQ3_M | PASS | done=true tools=9/9 drift=0 turns=5 trunc=false think=659c med=2277ms |
| 27B_Heretic_NEO_CODE_IQ4_XS | PASS | done=true tools=5/5 drift=0 turns=6 trunc=false think=607c med=2114ms |

_verdict PASS = scenario check passed. detail: tools=valid/total tool calls, drift=raw <tool_call> not structured, trunc=hit max_turns, think=reasoning chars burned, med=median request ms._

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-03 15:58:36 -0400
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
