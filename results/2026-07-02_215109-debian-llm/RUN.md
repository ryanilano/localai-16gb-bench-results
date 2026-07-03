# Benchmark run — 2026-07-02_215109-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_IQ4_XS | IQ4_XS | dense | 0 | 1737.096705 | 36.642655 | 15024 | 605 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 4096 | 1666.397366 | 35.780598 | 15164 | 923 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 8192 | 1600.450409 | 35.318794 | 15304 | 1071 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 16384 | 1438.541360 | 34.032229 | 15568 | 1353 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14254 | 445 | FAIL |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 0 | 821.615706 | 57.391260 | 2784 | 679 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 4096 | 804.334595 | 56.264975 | 2826 | 798 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 8192 | 792.276396 | 56.053169 | 2870 | 844 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 16384 | 778.090317 | 54.281050 | 2954 | 940 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 32768 | 762.609742 | 52.322749 | 3124 | 1129 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 0 | 472.876378 | 52.047828 | 2902 | 679 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 4096 | 669.493430 | 55.301633 | 2944 | 797 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 8192 | 649.265090 | 54.959133 | 2988 | 845 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 16384 | 636.111993 | 53.616215 | 3072 | 939 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 32768 | 642.936195 | 51.525861 | 3242 | 1127 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 0 | 839.738319 | 58.184446 | 2784 | 667 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 4096 | 860.102448 | 56.544445 | 2826 | 785 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 8192 | 827.978725 | 56.688533 | 2870 | 831 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 16384 | 820.227035 | 55.539553 | 2954 | 927 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 32768 | 826.734729 | 53.036729 | 3124 | 1116 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 0 | 1496.841404 | 36.453720 | 13102 | 734 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 4096 | 1450.280855 | 35.588068 | 13242 | 832 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 8192 | 1391.193161 | 34.886728 | 13364 | 973 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 16384 | 1273.887889 | 33.480246 | 13646 | 1254 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 32768 | 1070.531452 | 31.236309 | 14212 | 1819 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 0 | 1713.209608 | 37.173305 | 14678 | 529 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 4096 | 1647.902640 | 36.285094 | 14800 | 824 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 8192 | 1580.941979 | 35.809448 | 14940 | 966 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 16384 | 1426.078907 | 34.492520 | 15222 | 1247 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 32768 | 1177.825166 | 32.318054 | 15770 | 1813 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 0 | 1506.018315 | 35.372814 | 14056 | 1191 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 4096 | 1449.846652 | 34.463869 | 14196 | 832 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 8192 | 1397.863945 | 33.964536 | 14336 | 972 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 16384 | 1284.650872 | 32.680763 | 14600 | 1254 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 32768 | 1078.397281 | 30.615445 | 15166 | 1819 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1711.214366 | 36.517104 | 14966 | 869 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 4096 | 1645.295442 | 35.664984 | 15106 | 830 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 8192 | 1574.015452 | 35.207659 | 15264 | 971 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1432.639797 | 33.941346 | 15528 | 1252 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14256 | 449 | FAIL |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 0 | 1577.325323 | 40.867984 | 12724 | 1195 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 4096 | 1519.538385 | 39.846595 | 12864 | 831 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 8192 | 1459.100553 | 39.056742 | 13004 | 971 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 16384 | 1336.608466 | 37.417768 | 13268 | 1253 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 32768 | 1117.184211 | 34.701164 | 13816 | 1818 | OK |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-02 21:51:09 -0400
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
