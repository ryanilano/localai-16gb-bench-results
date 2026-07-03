# Benchmark run — 2026-07-03_005125-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 0 | 1461.872654 | 39.675674 | 12712 | 514 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 16384 | 1345.501726 | 37.601873 | 13238 | 1244 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 32768 | 1117.608171 | 34.878196 | 13822 | 1818 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 49152 | 926.894830 | 32.453881 | 14344 | 2387 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 65536 | 802.300666 | 30.310245 | 14888 | 2953 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 81920 | 706.262421 | 28.165158 | 15432 | 3517 | OK |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-03 00:51:25 -0400
hostname:             debian-llm

## NVIDIA driver / GPU
gpu (driver,name,vram): 595.71.05, NVIDIA GeForce RTX 4070 Ti SUPER, 16376 MiB
power (limit,max):      250.00 W, 285.00 W
driver userspace:       NVRM version: NVIDIA UNIX x86_64 Kernel Module  595.71.05  Fri Apr 24 06:36:12 UTC 2026
GCC version:  gcc version 14.2.0 (Debian 14.2.0-19) 

## CUDA toolkit
nvcc release:           Cuda compilation tools, release 13.3, V13.3.73
/usr/local/cuda ->:     /usr/local/cuda-13.3

## llama.cpp
resolved binary:        /root/llama.cpp/build-13.3/bin/llama-bench
source git commit:      4fc4ec5541b243957ae5099edb67372f8f3b550e
version output:
  version: 9859 (4fc4ec554)
  built with GNU 14.2.0 for Linux x86_64
```
