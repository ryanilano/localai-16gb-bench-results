# Benchmark run — 2026-07-02_175923-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_IQ4_XS | IQ4_XS | dense | 0 | 1724.926360 | 36.642602 | 15024 | 585 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 4096 | 1665.052372 | 35.917050 | 15134 | 898 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 8192 | 1587.465186 | 35.316649 | 15304 | 1052 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 16384 | 1439.121266 | 34.030293 | 15568 | 1348 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14254 | 442 | FAIL |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 0 | 632.628234 | 50.146413 | 2784 | 1047 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 4096 | 808.237228 | 52.044473 | 2826 | 790 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 8192 | 786.119635 | 51.687312 | 2870 | 836 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 16384 | 780.080877 | 50.596235 | 2954 | 933 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 32768 | 770.844022 | 48.480634 | 3124 | 1123 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 0 | 261.654990 | 50.359025 | 2902 | 1204 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 4096 | 663.376384 | 54.863113 | 2944 | 790 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 8192 | 657.810904 | 54.539155 | 2988 | 838 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 16384 | 641.635056 | 53.331969 | 3072 | 933 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 32768 | 635.519948 | 51.264645 | 3242 | 1121 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 0 | 629.707321 | 47.287517 | 2784 | 664 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 4096 | 860.190651 | 51.019634 | 2826 | 780 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 8192 | 831.395941 | 50.539024 | 2870 | 827 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 16384 | 829.348816 | 49.290011 | 2954 | 923 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 32768 | 806.768978 | 47.505299 | 3124 | 1112 | OK |
| 27B_Heretic_Youssofal | Q4_K_M | dense | 0 |  |  | 15310 | 689 | FAIL |
| 27B_Heretic_Youssofal | Q4_K_M | dense | 4096 |  |  | 15310 | 447 | FAIL |
| 27B_Heretic_Youssofal | Q4_K_M | dense | 8192 |  |  | 15310 | 447 | FAIL |
| 27B_Heretic_Youssofal | Q4_K_M | dense | 16384 |  |  | 15310 | 448 | FAIL |
| 27B_Heretic_Youssofal | Q4_K_M | dense | 32768 |  |  | 15310 | 447 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 0 | 1498.076302 | 36.396408 | 13102 | 531 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 4096 | 1445.696086 | 35.557338 | 13242 | 826 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 8192 | 1392.348816 | 34.878923 | 13382 | 967 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 16384 | 1275.064088 | 33.490375 | 13646 | 1250 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 32768 | 1069.354708 | 31.244088 | 14212 | 1814 | OK |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-02 17:59:23 -0400
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
