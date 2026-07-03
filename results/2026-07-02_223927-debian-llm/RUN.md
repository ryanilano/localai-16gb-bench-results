# Benchmark run — 2026-07-02_223927-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_IQ4_XS | IQ4_XS | dense | 0 | 1720.192316 | 36.605529 | 15024 | 1225 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 16384 | 1442.583884 | 34.011740 | 15568 | 1350 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14254 | 445 | FAIL |
| 27B_IQ4_XS | IQ4_XS | dense | 49152 |  |  | 14254 | 446 | FAIL |
| 27B_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14254 | 446 | FAIL |
| 27B_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14254 | 445 | FAIL |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 0 | 603.400024 | 52.926112 | 2784 | 678 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 32768 | 758.972577 | 52.245559 | 3124 | 1126 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 65536 | 591.703928 | 47.944194 | 3464 | 1502 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 131072 | 685.507739 | 41.846150 | 4202 | 2265 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 196608 | 627.615413 | 36.857354 | 5074 | 3030 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 261632 | 580.558513 | 32.946107 | 5938 | 3779 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 0 | 398.822802 | 53.388872 | 2902 | 1463 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 32768 | 635.831333 | 51.523286 | 3242 | 1124 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 65536 | 612.492919 | 47.767561 | 3582 | 1502 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 131072 | 580.512657 | 41.330779 | 4352 | 2265 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 196608 | 330.982673 | 35.791142 | 5224 | 3298 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 261632 | 248.543301 | 26.581345 | 6088 | 3853 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 0 | 471.060176 | 48.394809 | 2784 | 1676 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 32768 | 809.187307 | 52.516410 | 3124 | 1114 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 65536 | 777.166521 | 48.780028 | 3464 | 1492 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 131072 | 711.481401 | 42.275532 | 4194 | 2252 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 196608 | 658.209347 | 36.804633 | 5066 | 3016 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 261632 | 601.719694 | 33.023900 | 5930 | 3769 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 0 | 1474.281707 | 36.236025 | 13102 | 1148 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 16384 | 1274.892703 | 33.454715 | 13646 | 1252 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 32768 | 1071.822007 | 31.225759 | 14212 | 1817 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 49152 | 895.477694 | 29.248361 | 14734 | 2381 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 65536 | 781.918411 | 27.347531 | 15278 | 2945 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 81920 | 687.401616 | 25.531205 | 15822 | 3511 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 0 | 1712.690128 | 37.171034 | 14660 | 1149 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 16384 | 1428.803277 | 34.502865 | 15204 | 1246 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 32768 | 1179.204902 | 32.319273 | 15770 | 1810 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 49152 |  |  | 13950 | 444 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 65536 |  |  | 13950 | 444 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 81920 |  |  | 13950 | 445 | FAIL |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 0 | 1366.238097 | 33.860838 | 14056 | 533 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 16384 | 1285.640199 | 32.740787 | 14600 | 1252 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 32768 | 1079.250097 | 30.633193 | 15166 | 1816 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 49152 | 900.813224 | 28.763703 | 15688 | 2381 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 65536 |  |  | 13328 | 445 | FAIL |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 81920 |  |  | 13328 | 442 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1713.743019 | 36.521102 | 14984 | 1845 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1444.277891 | 33.957169 | 15510 | 1251 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14256 | 747 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 49152 |  |  | 14256 | 445 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 65536 |  |  | 14256 | 448 | FAIL |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 81920 |  |  | 14256 | 445 | FAIL |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 0 | 1578.039234 | 40.890601 | 12724 | 1131 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 16384 | 1339.515333 | 37.493069 | 13268 | 1250 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 32768 | 1117.792413 | 34.736411 | 13834 | 1814 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 49152 | 922.913862 | 32.423276 | 14356 | 2379 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 65536 | 804.581767 | 30.250219 | 14900 | 2944 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 81920 | 704.706324 | 28.128623 | 15444 | 3510 | OK |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-02 22:39:27 -0400
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
