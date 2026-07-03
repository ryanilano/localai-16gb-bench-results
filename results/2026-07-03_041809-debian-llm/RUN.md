# Quality run — 2026-07-03_041809-debian-llm

## Models

- 35B_UD-Q4_K_XL -> unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q4_K_XL  [ngl=99]
- 35B_UD-Q3_K_M -> unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q3_K_M  [ngl=99]

## Prompts

- 01_palindrome.txt
- 01_semver_compare.txt
- 02_debug_subtle.txt
- 02_refactor.txt
- 03_agent_plan.txt
- 03_refactor_constrained.txt
- 04_toolcall_plan.txt
- 05_strict_format.txt
- 06_security_dualuse.txt

## Provenance

```
# Benchmark run provenance
timestamp:            2026-07-03 04:18:09 -0400
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
resolved binary:        /root/llama.cpp/build-13.3/bin/llama-server
source git commit:      4fc4ec5541b243957ae5099edb67372f8f3b550e
version output:
  version: 9859 (4fc4ec554)
  built with GNU 14.2.0 for Linux x86_64
```
