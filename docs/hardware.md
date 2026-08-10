# Hardware and Software Environment

The v0.1 Research Preview was validated on a memory-constrained laptop.

## CPU

- Processor: Intel Core i7-8550U
- Physical cores: 4
- Hardware threads: 8
- Architecture: x86_64
- ISA features used by the CPU build include AVX2 and FMA
- ds4 worker threads used for the reported first-token experiments: 6

## Memory

- Physical RAM: approximately 7.7 GiB
- Swap file: approximately 12 GiB
- zram: approximately 1.9 GiB

The tested 78.62 GiB GGUF is approximately ten times larger than physical RAM.

## Storage

- Model storage: NVMe SSD
- Filesystem: ext4

Observed direct sequential I/O on the research machine was approximately:

- write: 624 MB/s
- read: 1.4-1.5 GB/s

These storage figures describe the test machine only. They are not minimum requirements and should not be generalized to other SSDs or filesystems.

## Operating System

- Ubuntu 22.04 LTS
- Linux kernel: 5.15 family
- CPU-only execution for the reported v0.1 validation
- No CUDA or GPU was used for the reported CPU result

## Virtual-Memory Configuration

The controlled cold first-token validation used:

- `vm.swappiness=10`
- zram reset to approximately zero used bytes before the run
- filesystem caches dropped before the run
- transparent huge pages enabled with defrag mode `madvise`

The v0.1 public result does not claim that these settings are optimal.

## Build Environment

The CPU candidate was built with:

```text
gcc 11.4
GNU Make 4.3
Python 3.10
git 2.34
```

The repository command used for the CPU build was:

```bash
make cpu
```

## Model Under Test

- Model: DeepSeek V4 Flash
- Architecture: `deepseek4`
- GGUF version: 3
- File size: 78.62 GiB
- Logical parameters: 284.33B
- Layers: 43
- Tensors: 1328
- AProjQ4 Q4_K tensors: 215

The model weights are not included in this repository.

## Reproducibility Boundary

The hardware description is provided so that results can be interpreted in context.

First-token latency, page-fault behavior, reclaim activity, and sustained generation behavior may change materially with CPU generation, RAM size, SSD performance, kernel version, swap configuration, filesystem state, and model quantization.
