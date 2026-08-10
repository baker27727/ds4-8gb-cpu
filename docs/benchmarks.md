# Benchmarks and Validation Results

## Scope

The primary v0.1 result is a functionality result:

A 78.62 GiB DeepSeek V4 Flash AProjQ4 GGUF completed the native whole-model CPU first-token diagnostic on a system with approximately 7.7 GiB of physical RAM.

The measurements below describe the tested machine and procedure. They are not universal performance claims.

## Controlled Cold First-Token Validation

Configuration:

- Upstream base: `84cc882`
- AProjQ4 CPU support: enabled
- Demand paging: enabled
- `DS4_CPU_NO_PREFETCH=1`
- CPU worker threads: 6
- `vm.swappiness=10`
- zram reset to approximately zero used bytes before the run
- filesystem caches dropped before the run
- prompt: `Hello`
- diagnostic: `--first-token-test`

Observed controlled run:

| Metric | Result |
|---|---:|
| Exit status | 0 |
| Wall-clock elapsed | 5.33 s |
| Maximum RSS | 6,204,632 KiB |
| Process major faults | 55,553 |
| Process minor faults | 96,819 |
| `/usr/bin/time -v` filesystem inputs | 12,787,656 |
| Process swaps | 0 |
| System `pgmajfault` delta | 4,637 |
| `pgscan_kswapd` delta | 0 |
| `pgscan_direct` delta | 0 |
| zram used after run | 0 B |

The diagnostic completed the native CPU path and emitted the expected first-token hidden-state and logits diagnostics.

The maximum RSS was below the physical-memory capacity of the test system while the mapped GGUF was 78.62 GiB.

This demonstrates execution under the tested conditions. It does not show that all prompts, context sizes, or sustained generation workloads will have the same memory footprint or latency.

## Selected-Expert Prefetch V4 Reproduction

An earlier research branch showed promising selected-expert prefetch behavior.

Before including that optimization in v0.1, the final V4 form was ported to the current upstream candidate and tested with four balanced cold-cache A/B pairs.

Results:

| Pair | V4 OFF | V4 ON | Change |
|---|---:|---:|---:|
| 1 | 5.550 s | 4.900 s | -11.71% |
| 2 | 4.860 s | 4.850 s | -0.21% |
| 3 | 4.910 s | 5.070 s | +3.26% |
| 4 | 4.860 s | 4.880 s | +0.41% |

Pooled means:

- V4 OFF mean: 5.045 s
- V4 ON mean: 4.925 s
- Mean change: -2.38%
- V4 wins: 2 of 4 pairs

Paired latency difference:

- Mean delta: -0.120 s
- 95% confidence interval: [-0.694, +0.454] s

The confidence interval crosses zero and V4 won only two of the four pairs.

Therefore the selected-expert V4 prefetch experiment is not included as a v0.1 performance optimization.

All eight extracted first-token result blocks from the balanced experiment were byte-identical.

## Why the Negative Result Is Included

A single earlier A/B comparison appeared substantially more favorable.

The balanced four-pair experiment did not reproduce that magnitude consistently.

The public release therefore reports the repeated balanced result and keeps V4 outside the v0.1 default code path.

## What Is Not Claimed

The v0.1 Research Preview does not claim:

- production generation speed,
- that the full model resides in RAM,
- that demand paging is always faster,
- that the tested VM configuration is optimal,
- or that the first-token result predicts sustained decode throughput.

The purpose of the v0.1 benchmark record is to document a reproducible functional boundary and to distinguish reproduced findings from inconclusive experiments.

## Later VM Research

Separate research on Linux reclaim, zram, and higher `vm.swappiness` values produced promising results on an earlier research branch.

Those results are intentionally not promoted into the v0.1 current-upstream benchmark claim. They should be rerun on the public candidate before being treated as release evidence.
