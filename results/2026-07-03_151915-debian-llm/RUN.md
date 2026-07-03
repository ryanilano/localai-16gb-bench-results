# Benchmark run — 2026-07-03_151915-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_IQ4_XS | IQ4_XS | dense | 0 | 1718.969584 | 36.538801 | 15016 | 620 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 16384 | 1436.268901 | 33.609694 | 15304 | 1094 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 32768 | 1183.536177 | 31.384316 | 15614 | 1403 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 49152 | 977.153368 | 29.342555 | 15880 | 1714 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14254 | 446 | FAIL |
| 27B_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14254 | 446 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 0 | 1721.434153 | 37.115512 | 14670 | 526 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 16384 | 1440.471115 | 34.087771 | 14942 | 987 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 32768 | 1183.742609 | 31.804850 | 15268 | 1296 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 49152 | 976.411808 | 29.711968 | 15534 | 1604 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 65536 | 840.429135 | 27.851387 | 15822 | 1913 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 81920 |  |  | 13950 | 446 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1713.292599 | 36.456706 | 14976 | 776 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1435.439675 | 33.544929 | 15248 | 992 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 | 1183.213327 | 31.337819 | 15574 | 1302 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 49152 | 976.825545 | 29.312375 | 15840 | 1609 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14256 | 446 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14256 | 447 | FAIL |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1696.614988 | 36.359035 | 14970 | 1103 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1435.830141 | 33.481535 | 15258 | 994 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 | 1181.831850 | 31.272113 | 15568 | 1302 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 49152 | 977.708211 | 29.248239 | 15834 | 1610 | OK |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14250 | 443 | FAIL |
| 27B_Heretic_NEO_CODE_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14250 | 443 | FAIL |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-03 15:19:15 -0400
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
