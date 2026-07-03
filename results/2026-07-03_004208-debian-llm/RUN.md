# Benchmark run — 2026-07-03_004208-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 0 | 1596.225183 | 41.125382 | 12712 | 527 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 4096 | 1531.045781 | 40.101898 | 12852 | 820 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 8192 | 1477.034381 | 39.285388 | 12992 | 969 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 16384 | 1346.998669 | 37.525700 | 13256 | 1251 | OK |
| 27B_Heretic_NEO_CODE_IQ3_M | IQ3_M | dense | 32768 | 1115.799499 | 34.831753 | 13822 | 1818 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1706.344806 | 36.434676 | 14978 | 722 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 4096 | 1644.216637 | 35.745199 | 15118 | 831 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 8192 | 1570.506622 | 35.124953 | 15258 | 972 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1422.929668 | 33.865443 | 15522 | 1256 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14250 | 448 | FAIL |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-03 00:42:08 -0400
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
