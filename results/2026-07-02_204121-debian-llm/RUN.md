# Benchmark run — 2026-07-02_204121-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_IQ4_XS | IQ4_XS | dense | 0 | 1728.888247 | 36.642911 | 15024 | 602 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 4096 | 1654.183424 | 35.789285 | 15164 | 919 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 8192 | 1590.056441 | 35.318438 | 15304 | 1066 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 16384 | 1433.819519 | 34.034362 | 15538 | 1349 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14254 | 443 | FAIL |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 0 | 595.414311 | 47.723118 | 2784 | 676 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 4096 | 699.662591 | 49.584011 | 2826 | 792 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 8192 | 455.882619 | 44.485561 | 2870 | 840 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 16384 | 702.658023 | 53.462532 | 2954 | 935 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 32768 | 761.280536 | 52.433486 | 3124 | 1125 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 0 | 455.444925 | 52.847149 | 2902 | 676 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 4096 | 669.659055 | 55.245422 | 2944 | 793 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 8192 | 647.927391 | 55.056346 | 2988 | 840 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 16384 | 608.563417 | 53.752064 | 3072 | 935 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 32768 | 635.724266 | 51.620723 | 3242 | 1122 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 0 | 795.247108 | 57.405808 | 2784 | 1120 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 4096 | 859.102741 | 57.194385 | 2826 | 778 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 8192 | 832.555402 | 56.454610 | 2870 | 827 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 16384 | 820.477911 | 54.650596 | 2954 | 921 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 32768 | 821.013301 | 53.072103 | 3124 | 1112 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 0 | 1498.715561 | 36.467357 | 13102 | 530 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 4096 | 1444.988969 | 35.482186 | 13242 | 826 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 8192 | 1390.079385 | 34.885990 | 13382 | 967 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 16384 | 1273.522277 | 33.500164 | 13646 | 1249 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 32768 | 1069.920091 | 31.229409 | 14212 | 1814 | OK |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 0 |  |  | 2 | 225 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 4096 |  |  | 2 | 226 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 8192 |  |  | 2 | 232 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 16384 |  |  | 2 | 228 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 32768 |  |  | 2 | 229 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 0 |  |  | 2 | 228 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 4096 |  |  | 2 | 227 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 8192 |  |  | 2 | 226 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 16384 |  |  | 2 | 228 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 32768 |  |  | 2 | 303 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 0 | 1719.489880 | 37.191141 | 14678 | 699 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 4096 | 1650.283488 | 36.430965 | 14800 | 820 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 8192 | 1580.400207 | 35.804334 | 14958 | 962 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 16384 | 1430.456905 | 34.492331 | 15204 | 1244 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 32768 | 1176.595526 | 32.318434 | 15788 | 1808 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 0 | 1506.308576 | 35.382025 | 14056 | 1186 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 4096 | 1454.019786 | 34.468514 | 14196 | 826 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 8192 | 1400.041557 | 33.948255 | 14336 | 967 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 16384 | 1284.661169 | 32.691671 | 14600 | 1249 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 32768 | 1079.108138 | 30.629029 | 15166 | 1814 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1715.680042 | 36.517419 | 14984 | 1522 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 4096 | 1648.028612 | 35.666308 | 15106 | 824 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 8192 | 1571.638180 | 35.205443 | 15264 | 965 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1425.668214 | 33.940079 | 15528 | 1248 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14256 | 442 | FAIL |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 0 | 1407.325017 | 40.666244 | 12724 | 1682 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 4096 | 1514.389661 | 39.846314 | 12864 | 823 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 8192 | 1459.067702 | 39.088005 | 13004 | 966 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 16384 | 1332.300600 | 37.439419 | 13268 | 1248 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 32768 | 1111.977180 | 34.740825 | 13834 | 1813 | OK |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-02 20:41:21 -0400
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
