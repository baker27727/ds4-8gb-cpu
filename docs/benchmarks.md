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

Historical follow-up experiments investigated Linux reclaim, zram, and `vm.swappiness` under the memory-constrained CPU/NVMe workload.

The balanced V16.1 comparison of swappiness 10 versus 100 produced a strong historical signal: pooled steady decode improved by 9.11% with 4/4 paired wins.

V16.2 compared swappiness 100 versus 150. The mean direction favored 150 by 1.60%, with 3/4 paired wins, but the paired 95% confidence interval crossed zero. That result is classified as `INCONCLUSIVE`.

V16.3 compared swappiness 100 versus 200. Swappiness 200 regressed sustained decode by 71.60% on average and lost all four balanced pairs. The paired confidence interval excluded zero. That result is classified as `STRONG-HISTORICAL`, with swappiness 200 rejected for the tested system and workload.

V16.4 compared swappiness 150 versus 175 using a strengthened full swap-tier reset before every run. Swappiness 175 reduced mean steady-decode latency by 2.14% and won all four balanced pairs, but the paired 95% confidence interval crossed zero. The result is classified as `INCONCLUSIVE`. The experiment brackets the observed degradation boundary between swappiness 175 and 200 without establishing an optimum.

V16.5 narrowed the upper region by comparing swappiness 175 versus 188 using the same full swap-tier reset. Swappiness 188 was 2.24% slower on mean steady decode and lost all four balanced pairs, but the paired 95% confidence interval again crossed zero. The result is classified as `INCONCLUSIVE`. Together with V16.4, this places the current candidate turnover region approximately between swappiness 175 and 188 without establishing 175 as an optimum.

V16.6 compared swappiness 175 versus 182 under the same strengthened reset methodology. Mean steady-decode latency differed by only 0.49%, with each setting winning two of four balanced pairs and a wide paired confidence interval crossing zero. The result is classified as `INCONCLUSIVE` and effectively neutral. Together with V16.5, the current degradation-onset bracket is narrowed approximately to swappiness 182 through 188 for the tested workload.

V16.7 compared swappiness 182 versus 185 with a fully clean pre-run swap baseline in all eight runs. Sustained decode remained effectively neutral: swappiness 185 was 0.92% slower on the mean, with a 2/4 versus 2/4 pair split and a wide confidence interval crossing zero. First decode showed a separate strong unfavorable signal at 185, with 182 winning all four pairs and the paired confidence interval excluding zero. The primary sustained-decode classification remains `INCONCLUSIVE`. The current sustained-degradation onset bracket is narrowed approximately to swappiness 185 through 188.

V16.8R repeated the swappiness 185 versus 187 comparison with an explicit pre-run swap qualification gate of 16 MiB. Attempts above the gate were rejected and retried. Accepted runs remained effectively neutral for sustained decode: swappiness 187 was 1.04% slower on the mean, with a 2/4 versus 2/4 pair split and a wide paired confidence interval crossing zero. VM and PSI behavior was also broadly similar. The result is classified as `INCONCLUSIVE`. The candidate sustained-degradation onset bracket is now approximately swappiness 187 through 188, pending a direct 187-versus-188 comparison.

Taken together, the historical evidence shows an effectively neutral sustained-decode region from approximately swappiness 175 through 187, an unfavorable but inconclusive sustained trend at 188, and a strong regression at 200. The current candidate sustained-degradation onset bracket is approximately 187 to 188 for the tested workload, pending direct confirmation.

These experiments reset zram and dropped filesystem caches, but V16.2 and V16.3 did not fully clear the disk-backed swapfile between every run. They remain historical research evidence and are not promoted into the current v0.1 public performance claim.

See `docs/evidence-register.md` entries E-010, E-013, E-014, E-015, E-016, E-017, E-018, E-019, and E-020 for classification and claim boundaries.

V16.9 directly compared swappiness 187 versus 188 using six balanced pairs and the qualified pre-run swap gate. All twelve accepted runs passed on attempt 1. Swappiness 188 was slower in five of six sustained-decode pairs, with a pooled mean regression of 1.19%, but the paired 95% confidence interval crossed zero. VM measurements also showed a small unfavorable tendency at 188, including higher file refaults, kswapd steal, and memory PSI. The result is classified as `INCONCLUSIVE` with a consistent mild unfavorable trend at 188.

With V16.9, fine-grained swappiness tuning is considered complete for this workload. The controlled evidence supports an effectively flat sustained-decode region from approximately 175 through 187, a mild unfavorable tendency at 188, and a strong demonstrated sustained regression at 200. Swappiness 187 is retained as a conservative working setting for subsequent experiments, not as a universal optimum.

### V17.4F — Early Hash Gate/Up Prefetch

V17 moved the research focus from Linux VM micro-tuning to sustained generation throughput.

V17.4F evaluated an opt-in early Gate/Up page-in hint for the first three hash-routed layers. The mechanism used token-known expert IDs to issue selected `WILLNEED` hints before attention, while preserving model arithmetic and generated output.

The final experiment used eight counterbalanced A/B pairs for sixteen valid runs. All runs passed output parity and produced 39 decode evaluations.

Mean sustained-decode latency was:

- Control A: `3226.417 ms/token`
- Variant B: `3215.474 ms/token`
- Nominal B vs A change: `-0.34%`

Variant B won `5/8` pairs. The paired mean difference was `-10.942 ms`, with a 95% confidence interval of `[-48.641, +26.756] ms`.

Because the confidence interval crosses zero, the result is classified as `INCONCLUSIVE`: no statistically established sustained-decode speedup was demonstrated.

Secondary diagnostics moved modestly in the expected direction:

- filesystem input activity: `-1.07%`
- major page faults: `-2.05%`

These are diagnostic signals only and do not establish a throughput improvement without a corresponding statistically supported latency reduction.

Run-position balance in the final protocol was good: position 2 was only `+0.28%` slower than position 1.

V17.4 is therefore closed without promoting the experimental hash-only mechanism as a performance optimization. The next research phase targets the later activation-dependent top-k routed layers, which represent the substantially larger remaining routed-expert workload.

See `docs/evidence-register.md` entry E-021 and `benchmarks/v17.4f-hash-early-gu-ab.csv` for the evidence and sanitized numeric data.


## V18 Runtime Winner Selection

V18 moved from hash-only early prefetch to activation-dependent routed
expert scheduling and then closed the main runtime-selection axes for the
tested CPU/NVMe host.

The retained research configuration is:

- early activation-dependent AIO Top-K: `2`
- residual Gate/Up: granular/direct consumption
- Down AIO budget: `6`
- CPU worker threads: `8`

These are historical research-branch results pending reproduction on the
public source candidate. They are not promoted here as universal or
current-public-branch performance claims.

### Top-K Selection

K1 was slower than K2 in all four paired comparisons, with a paired
K1-minus-K2 95% confidence interval of `[+0.544, +2.766] s`.

K3 and layer-masked K3 produced small favorable mean directions relative
to K2, but both paired confidence intervals crossed zero. K2 was retained
because neither K3 variant demonstrated a latency advantage sufficient
to justify additional speculative I/O.

See E-022 and `benchmarks/v18.10-topk-selection.csv`.

### Residual Gate/Up Alternatives

The no-allocation two-batch alternative was approximately `10.19%`
slower than the retained granular/direct baseline and lost all four
pairs. Its paired 95% confidence interval was
`[+3.507, +6.758] s`.

Micro-batch=2 also lost all four observed pairs, but its confidence
interval crossed zero, so that result remains inconclusive rather than a
statistically established regression.

See E-023 and `benchmarks/v18.10-residual-gu.csv`.

### Down AIO Budget

Mean elapsed time was:

- D4: `53.060 s`
- D5: `52.403 s`
- D6: `51.663 s`

D6 beat both smaller budgets in all six paired comparisons, and both
paired confidence intervals excluded zero. Down AIO budget 6 was
therefore retained.

See E-024 and `benchmarks/v18.11c-down-aio-budget.csv`.

### CPU Thread Count

The initial thread sweep strongly favored T8 over the former T6
baseline. A focused T8-versus-T4 confirmation then produced a
`1.573%` mean reduction with `9/10` paired wins and a 95% confidence
interval of `[-1.302, -0.268] s`.

The first sustained campaign was unresolved. A subsequent qualified
sustained repetition produced:

- T4: `132.772 s`
- T8: `130.357 s`
- T8 reduction: `1.819%`
- T8 wins: `6/6`
- paired 95% confidence interval: `[-3.436, -1.394] s`

The laptop remained thermally constrained and T8 incurred higher measured
thermal/throttle cost. The result is therefore reported as paired
sustained evidence under a thermally constrained host, not as a
constant-temperature benchmark.

See E-025 and `benchmarks/v18.12-thread-selection.csv`.
