# Benchmark run — 2026-07-02_193446-debian-llm

## Results

| label | quant | type | depth | pp_tok_s | tg_tok_s | vram_peak_mib | ram_used_peak_mib | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 27B_IQ4_XS | IQ4_XS | dense | 0 | 1734.981027 | 36.633473 | 15024 | 1132 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 4096 | 1668.892626 | 35.794946 | 15134 | 911 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 8192 | 1595.106550 | 35.324831 | 15304 | 1061 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 16384 | 1438.490717 | 34.034927 | 15568 | 1346 | OK |
| 27B_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14254 | 439 | FAIL |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 0 | 345.942789 | 48.712057 | 2784 | 23812 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 4096 | 578.508577 | 52.621994 | 2826 | 826 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 8192 | 740.426455 | 55.167741 | 2870 | 1242 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 16384 | 777.255806 | 54.531522 | 2954 | 937 | OK |
| 35B_UD-IQ4_NL_XL | UD-IQ4_NL_XL | moe | 32768 | 761.377606 | 52.637046 | 3124 | 1125 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 0 | 615.713037 | 56.221581 | 2902 | 1252 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 4096 | 661.667531 | 55.401890 | 2944 | 793 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 8192 | 648.366981 | 54.991779 | 2988 | 840 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 16384 | 650.366855 | 53.814144 | 3072 | 936 | OK |
| 35B_UD-Q4_K_XL | UD-Q4_K_XL | moe | 32768 | 642.080128 | 51.721420 | 3242 | 1125 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 0 | 869.373593 | 58.509431 | 2784 | 665 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 4096 | 859.856698 | 56.816079 | 2826 | 782 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 8192 | 835.679286 | 56.921139 | 2870 | 830 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 16384 | 820.040433 | 55.399804 | 2954 | 926 | OK |
| 35B_UD-Q3_K_M | UD-Q3_K_M | moe | 32768 | 809.495065 | 51.649477 | 3124 | 1115 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 0 | 1498.689891 | 36.477550 | 13102 | 1163 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 4096 | 1447.110977 | 35.512403 | 13242 | 830 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 8192 | 1392.692946 | 34.906324 | 13382 | 970 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 16384 | 1275.691666 | 33.521018 | 13646 | 1253 | OK |
| 27B_Heretic_Youssofal_Q3_K_M | Q3_K_M | dense | 32768 | 1070.948715 | 31.232698 | 14212 | 1817 | OK |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 0 |  |  | 2 | 228 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 4096 |  |  | 2 | 228 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 8192 |  |  | 2 | 227 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 16384 |  |  | 2 | 228 | FAIL |
| 27B_Heretic_Youssofal_Q3_K_L | Q3_K_L | dense | 32768 |  |  | 2 | 229 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 0 |  |  | 2 | 231 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 4096 |  |  | 2 | 229 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 8192 |  |  | 2 | 229 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 16384 |  |  | 2 | 229 | FAIL |
| 35B_Heretic_HauhauCS | Q4_K_P | moe | 32768 |  |  | 2 | 231 | FAIL |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 0 | 1715.014318 | 37.188416 | 14678 | 1438 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 4096 | 1645.409058 | 36.456000 | 14818 | 823 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 8192 | 1579.213949 | 35.810987 | 14958 | 965 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 16384 | 1429.091255 | 34.499847 | 15222 | 1247 | OK |
| 27B_HauhauCS_Balanced | IQ4_XS | dense | 32768 | 1185.129188 | 32.324083 | 15788 | 1812 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 0 | 1504.440972 | 35.349195 | 14056 | 1767 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 4096 | 1451.225924 | 34.458703 | 14196 | 830 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 8192 | 1401.697973 | 33.979456 | 14336 | 971 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 16384 | 1291.414552 | 32.695503 | 14600 | 1253 | OK |
| 27B_HauhauCS_Balanced_Q3_K_P | Q3_K_P | dense | 32768 | 1074.654048 | 30.605380 | 15166 | 1817 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 0 | 1712.116618 | 36.525185 | 14984 | 532 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 4096 | 1641.695149 | 35.795283 | 15124 | 828 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 8192 | 1574.931611 | 35.212992 | 15264 | 969 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 16384 | 1430.727243 | 33.939736 | 15510 | 1252 | OK |
| 27B_NEO_CODE_IQ4_XS | IQ4_XS | dense | 32768 |  |  | 14256 | 449 | FAIL |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 0 | 1418.767573 | 39.419911 | 12724 | 15157 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 4096 | 1528.429086 | 39.861788 | 12864 | 829 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 8192 | 1463.347104 | 39.124237 | 13004 | 970 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 16384 | 1334.130690 | 37.433796 | 13268 | 1252 | OK |
| 27B_NEO_CODE_IQ3_M | IQ3_M | dense | 32768 | 1117.247405 | 34.720926 | 13816 | 1817 | OK |

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-02 19:34:47 -0400
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
