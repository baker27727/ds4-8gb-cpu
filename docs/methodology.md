# Research Methodology

## Objective

The v0.1 Research Preview investigates a narrow question:

Can a quantized DeepSeek V4 Flash GGUF that is substantially larger than physical RAM execute a real CPU whole-model path on a memory-constrained Linux system using mmap-backed NVMe storage?

The work separates functional validation from performance experiments.

## Public Code Baseline

The public candidate is based on upstream ds4 commit `84cc882`.

Two code changes are included above that baseline:

1. CPU support for AProjQ4 dense attention projections.
2. Opt-in CPU demand paging through `DS4_CPU_NO_PREFETCH`.

Experimental selected-expert prefetch work is not included in the v0.1 release candidate.

## Model Under Test

The tested GGUF reports:

- Model: DeepSeek V4 Flash
- Architecture: `deepseek4`
- GGUF: v3
- Layers: 43
- Tensors: 1328
- Logical parameters: 284.33B
- File size: 78.62 GiB

The tested AProjQ4 configuration contains 215 Q4_K tensors. Model weights are not distributed with this repository.

## Hardware Class

Primary test system:

- Intel Core i7-8550U
- 4 cores / 8 threads
- approximately 7.7 GiB physical RAM
- NVMe SSD
- Ubuntu 22.04
- Linux 5.15 family
- CPU-only execution

Six ds4 CPU worker threads were used for the reported first-token experiments.

## Functional Acceptance

The public candidate was considered functionally valid only after:

1. `make cpu` completed successfully.
2. `--inspect` recognized the real AProjQ4 GGUF.
3. `DS4_CPU_NO_PREFETCH=1` was observed on the CPU execution path.
4. `--first-token-test` completed through the native CPU whole-model path.
5. The process returned exit status 0.

## Demand-Paging Mode

When `DS4_CPU_NO_PREFETCH=1` is enabled, the CPU path skips the whole-model `POSIX_MADV_WILLNEED` request. The GGUF remains mmap-backed and pages are brought into memory on demand.

The default upstream behavior remains unchanged when the environment variable is not enabled.

## Controlled Cold-State Procedure

For controlled first-token experiments:

1. Keep `vm.swappiness=10`.
2. Reset `/dev/zram0` so it starts at approximately zero used bytes.
3. Run `sync`.
4. Drop filesystem caches through `/proc/sys/vm/drop_caches`.
5. Use the same GGUF and binary.
6. Use six CPU threads.
7. Use the prompt `Hello`.
8. Run `--first-token-test`.
9. Capture `/usr/bin/time -v` and selected `/proc/vmstat` counters.

Cache dropping and swap manipulation require elevated privileges and can affect other processes.

## Metric Interpretation

Process metrics from `/usr/bin/time -v` and system-wide counters from `/proc/vmstat` are treated separately. A process major-fault count should not be assumed to equal the system-wide `pgmajfault` delta.

## A/B Correctness Control

During the selected-expert V4 prefetch reproduction experiment, the first-token numerical result block from each run was compared byte-for-byte. All eight outputs from the balanced four-pair experiment were identical.

The performance result itself was inconclusive, so that optimization was removed from the v0.1 release candidate.

## Reporting Policy

A favorable single run is not promoted to a release performance claim. Repeated and balanced comparisons are preferred, and experiments that do not reproduce consistently are documented as inconclusive.

The principal v0.1 claim is functional: a 78.62 GiB DeepSeek V4 Flash AProjQ4 GGUF completed a native whole-model CPU first-token diagnostic on the tested system with approximately 7.7 GiB of physical RAM.


## V18 Throughput Follow-Up Methodology

Later V18 research evaluates activation-dependent routed-expert I/O and
CPU runtime scheduling separately from the original v0.1 first-token
functional claim.

The V18 comparison policy is:

1. compare candidates using paired, balanced, or rotated run order where
   the experiment supports it;
2. preserve the same model and workload within each comparison;
3. require output parity before interpreting performance;
4. treat elapsed-time differences as the primary performance signal;
5. report paired win count, paired mean difference, and 95% confidence
   interval when available;
6. treat page faults, filesystem input, device-read volume, RSS, CPU
   time, and thermal counters as diagnostic signals rather than direct
   throughput claims;
7. do not promote a favorable mean when the paired confidence interval
   crosses zero;
8. retain simpler/lower-speculation behavior when candidate performance
   is statistically tied and the more aggressive variant adds I/O.

The repeated short-run V18.10/V18.12 comparisons produced the output
SHA-256:

`b6cf063007cd61da094a9bf5000df2ff8ef74ba74efe658a16fd6d2de986d81f`

The sustained V18.12 comparisons used:

`f38d5456e87b8000941cbb1f6e5090d0da47a1548515e9b42885bbf67e6d2646`

### Thermal Qualification

Sustained laptop measurements are especially sensitive to CPU
temperature, frequency limits, and hardware throttling.

The V18.12E2 repetition reduced run-to-run variability sufficiently to
produce a paired T8 advantage whose confidence interval excluded zero,
but the host still exhibited material thermal throttling and T8 carried
higher measured thermal/throttle counters.

Accordingly, the evidence is described as paired sustained evidence
under a thermally constrained host. It is not described as a strict
constant-temperature benchmark, and it is not generalized to other CPU
topologies or cooling systems.

### Public-Evidence Boundary

Until the protected V18 source candidate is separately promoted, built,
and parity-tested from the clean publication branch, E-022 through E-025
remain research-branch evidence.

Evidence publication and source promotion are intentionally separate
steps.
