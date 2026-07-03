# Benchmark run — 2026-07-03_151252-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_IQ4_XS | IQ4_XS | dense | 0 | 1730.380875 | 36.636460 | 15024 | 1068 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 16384 | 1446.719179 | 33.933202 | 15568 | 1313 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14254 | 406 | FAIL |
| 27B_IQ4_XS | IQ4_XS | dense | 49152 |  |  | 14254 | 410 | FAIL |
| 27B_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14254 | 414 | FAIL |
| 27B_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14254 | 418 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 0 | 1724.727656 | 37.208716 | 14678 | 519 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 16384 | 1439.653844 | 34.403907 | 15222 | 1240 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 32768 | 1187.483233 | 32.328732 | 15788 | 1805 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 49152 |  |  | 13950 | 440 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 65536 |  |  | 13950 | 441 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 81920 |  |  | 13950 | 441 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1306.092995 | 33.675725 | 14984 | 530 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1437.023214 | 33.657730 | 15528 | 1249 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14256 | 444 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 49152 |  |  | 14256 | 445 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14256 | 445 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14256 | 445 | FAIL |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1299.752567 | 33.230559 | 14978 | 1927 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1435.417333 | 33.748214 | 15522 | 1249 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14250 | 444 | FAIL |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 49152 |  |  | 14250 | 444 | FAIL |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14250 | 444 | FAIL |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14250 | 444 | FAIL |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-03 15:12:52 -0400
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
resolved binary:        /root/llama.cpp/build-13.3/bin/llama-bench
source git commit:      4fc4ec5541b243957ae5099edb67372f8f3b550e
version output:
  version: 9859 (4fc4ec554)
  built with GNU 14.2.0 for Linux x86_64
```
