# Research Evidence Register

This register separates verified public-release evidence from historical research results and pending evidence extraction.

## Evidence Status Definitions

- `VERIFIED-PUBLIC`: reproduced on the current public v0.1 candidate.
- `CONTROLLED`: measured with a defined controlled A/B or cold-state procedure.
- `STRONG-HISTORICAL`: strong repeated result from the research branch, not yet reproduced on the public candidate.
- `INCONCLUSIVE`: measured effect did not reproduce reliably or confidence interval includes zero.
- `REGRESSION`: controlled experiment made the target metric worse.
- `PENDING-EXTRACTION`: source logs exist but have not yet been converted into sanitized public evidence.

## Evidence Index

| ID | Experiment | Status | Release relevance |
|---|---|---|---|
| E-001 | 8GB demand-paged CPU functional validation | FUNCTIONAL-HISTORICAL | Core v0.1 |
| E-002 | AProjQ4 CPU validation and Q8 to Q4 comparison | CONTROLLED | Core v0.1 |
| E-003 | V1 selected-expert prefetch | INCONCLUSIVE | Historical |
| E-004 | V2 select-once expert reuse | INCONCLUSIVE | Historical |
| E-005 | V3 staged Gate+Up / Down prefetch | INCONCLUSIVE | Historical |
| E-006 | V4 expert-major Gate+Up prefetch | INCONCLUSIVE | Historical |
| E-007 | V5-V13 profiling investigation | DIAGNOSTIC-HISTORICAL | Historical |
| E-008 | V14 shared Gate+Up residency | INCONCLUSIVE | Historical |
| E-009 | V15 shared Down residency | INCONCLUSIVE | Historical |
| E-010 | V16.1 swappiness 10 vs 100 | STRONG-HISTORICAL | Follow-up candidate |
| E-011 | V4 latest-upstream reproduction | INCONCLUSIVE | Rejected for v0.1 |
| E-012 | Public v0.1 controlled cold first-token validation | VERIFIED-PUBLIC | Core v0.1 |
| E-013 | V16.2 swappiness 100 vs 150 | INCONCLUSIVE | Historical follow-up |
| E-014 | V16.3 swappiness 100 vs 200 | STRONG-HISTORICAL | Historical follow-up |
| E-015 | V16.4 swappiness 150 vs 175 with full swap reset | INCONCLUSIVE | Historical follow-up |
| E-016 | V16.5 swappiness 175 vs 188 with full swap reset | INCONCLUSIVE | Historical follow-up |
| E-017 | V16.6 swappiness 175 vs 182 with full swap reset | INCONCLUSIVE | Historical follow-up |

## E-010 — Linux VM Swappiness 10 vs 100

### Classification

- Status: `STRONG-HISTORICAL`
- Experiment type: controlled balanced A/B
- Pairs: 4
- A condition: `vm.swappiness=10`
- B condition: `vm.swappiness=100`
- Steady decode window: evals 5-19
- Research version: `da251ac093006ad82599a47d0dab4fa16c55f3ad`
- Research tag: `cpu-v15-shared-down-causal-success`
- Public release status: not yet reproduced on the current v0.1 public branch

### Decode Latency

| Pair | A10 | B100 | Change |
|---|---:|---:|---:|
| 1 | 3615.51 ms | 3273.43 ms | -9.46% |
| 2 | 3598.33 ms | 3336.19 ms | -7.28% |
| 3 | 3633.76 ms | 3333.51 ms | -8.26% |
| 4 | 3731.51 ms | 3308.19 ms | -11.34% |
| Pooled | 3644.78 ms | 3312.83 ms | -9.11% |

- B100 wins: 4/4 pairs
- Mean paired delta: -331.95 ms
- 95% CI: [-441.92, -221.97] ms

### Memory and I/O Effects

| Metric | B100 vs A10 |
|---|---:|
| Major faults | -26.57% |
| Filesystem input | -26.42% |
| File refaults | -31.25% |
| File pages stolen | -25.81% |
| kswapd pages stolen | -25.64% |
| Swap-in | +441.27% |
| PSI some total delta | -33.48% |
| PSI full total delta | -33.61% |
| Mean zram footprint | +55.29% |

### Output Parity

All eight captured output files were byte-identical.

```text
SHA256: b6cf063007cd61da094a9bf5000df2ff8ef74ba74efe658a16fd6d2de986d81f
Files compared: 8
Size per output: 73 bytes
Identical: 8/8
```

### Interpretation

Increasing swappiness from 10 to 100 coincided with substantially more compressed-swap use and substantially less file-backed model refault and NVMe input. Under the tested configuration this was associated with lower decode latency and lower memory PSI.

This is evidence from the historical research branch. It is not yet a public-v0.1 performance claim.

Sanitized numeric source:

`benchmarks/v16-swappiness-summary.csv`


## E-001 — Original 8GB Demand-Paged CPU Functional Validation

### Classification

- Status: `FUNCTIONAL-HISTORICAL`
- Purpose: establish that a quantized model substantially larger than physical RAM can execute through the native CPU path using mmap-backed demand paging from NVMe.
- Source research commit: `d6823df`
- Public-release relationship: historical foundation for the opt-in demand-paging feature; performance numbers below are not measurements from the current public v0.1 branch.

### Model Under Test

| Property | Value |
|---|---:|
| Model | DeepSeek V4 Flash |
| Architecture | `deepseek4` |
| GGUF version | 3 |
| File size | 80.76 GiB |
| Logical parameters | 284.33B |
| Layers | 43 |
| Tensors | 1328 |
| Q8_0 tensors | 345 |
| Q2_K tensors | 43 |
| IQ2_XXS tensors | 86 |

The runtime reported `DS4_CPU_NO_PREFETCH=1` as active through the message:

`ds4: CPU whole-model prefetch disabled; using demand-paged NVMe weights`

### First-Token Whole-Model CPU Pass

| Metric | Result |
|---|---:|
| Wall-clock elapsed | 6.35 s |
| Maximum RSS | 6,096,700 KiB (~5.81 GiB) |
| Major page faults | 77,704 |
| Minor page faults | 129,603 |
| Filesystem inputs | 17,022,288 |
| Process swaps | 0 |
| Exit status | 0 |

The diagnostic emitted `final_hc`, logits, and top-logit output after the first-token whole-model CPU pass.

### Multi-Token Generation

A four-token generation test completed successfully.

| Metric | Result |
|---|---:|
| Prefill | 0.15 t/s |
| Generation | 0.20 t/s |
| Wall-clock elapsed | 26.52 s |
| Maximum RSS | 6,429,488 KiB (~6.13 GiB) |
| Major page faults | 330,246 |
| Filesystem inputs | 72,818,752 |
| Process swaps | 0 |
| Exit status | 0 |

This test is used as evidence of multi-token execution, not as a quality benchmark.

### Prompt-Specific Chat Validation

Prompt:

`What is the capital of Norway? Reply with only the city name.`

Generated answer: `Oslo`

The generated stdout fragments were interleaved with diagnostic stderr messages in the captured file, but reconstruct to `Oslo`.

| Metric | Result |
|---|---:|
| Prefill | 0.49 t/s |
| Generation | 0.15 t/s |
| Wall-clock elapsed | 60.12 s |
| Maximum RSS | 6,398,032 KiB (~6.10 GiB) |
| Major page faults | 403,488 |
| Filesystem inputs | 87,191,448 |
| Process swaps | 0 |
| Exit status | 0 |

### Source Evidence Hashes

```text
deepseek-inspect.txt      3d1287fff8dde269dd1d753b7cb71347832f219f5a05655f5cc52530035fe531
first-token-8gb.txt       995f33b513763ed16084caadf2760b7e059c6e33f8dbe48e8aa6b90136b02f55
real-generation-8gb.txt  dec21fd981b206dc029412c3d12403adceb202065463302c4b656137c4bac68b
norway-chat-8gb.txt       6ee34fa7cca7be442f56af282c00fa7ba84c97e22d50a5696bca438504e6af39
```

### Claim Boundary

This experiment demonstrates functional CPU execution of an 80.76 GiB quantized GGUF on the tested memory-constrained system through demand-paged NVMe-backed weights. It does not mean the complete model fits in physical RAM, and its latency values are not presented as current public-v0.1 performance results.


## E-002 — AProjQ4 CPU Attention-Projection Quantization

### Classification

- Status: `CONTROLLED`
- Functional validation: historical research branch
- Comparison type: single controlled cold Q8-versus-Q4 comparison
- Release relationship: AProjQ4 CPU support is included in the v0.1 public candidate, but the historical performance numbers below are not presented as current-public-branch benchmarks.
- Original AProjQ4 source commit: `3b57f9b94d37688a63343475ea6b8402844a1dd2`
- Original author: `oppo99 <oppo.giorgio@live.com>`

### Model Transformation

| Property | AProjQ8 | AProjQ4 |
|---|---:|---:|
| Exact file bytes | 86,720,111,488 | 84,420,584,032 |
| Approximate size | 80.76 GiB | 78.62 GiB |
| Q8_0 tensors | 345 | 130 |
| Q4_K tensors | 0 | 215 |
| Logical parameters | 284.33B | 284.33B |
| Total tensors | 1328 | 1328 |

The conversion changed 215 dense attention-projection tensors from Q8_0 to Q4_K. The resulting GGUF was smaller by 2,299,527,456 bytes, approximately 2.142 GiB or 2.65% of the original file size.

Examples in the conversion log show `attn_kv`, `attn_output_a`, `attn_output_b`, `attn_q_a`, and `attn_q_b` changing from Q8_0 to Q4_K while sampled shared-expert tensors and `output.weight` remained Q8_0.

### Controlled Cold Q8 vs Q4 Comparison

Both runs used CPU execution, six worker threads, demand paging through `DS4_CPU_NO_PREFETCH=1`, the same short prompt, and zero process swaps.

| Metric | AProjQ8 | AProjQ4 | Change |
|---|---:|---:|---:|
| First decode eval | 6365.832 ms | 4108.339 ms | -35.46% |
| Whole-process elapsed | 13.83 s | 9.61 s | -30.51% |
| Maximum RSS | 6,849,544 KiB | 6,892,140 KiB | +0.62% |
| Major page faults | 155,150 | 101,975 | -34.27% |
| Minor page faults | 272,667 | 175,528 | -35.63% |
| Filesystem inputs | 36,382,152 | 23,405,072 | -35.67% |
| Process swaps | 0 | 0 | unchanged |

The 35.46% first-decode latency reduction corresponds to approximately 54.95% higher reciprocal first-decode throughput. This must not be interpreted as a 54.95% improvement in general sustained generation throughput.

### AProjQ4 Multi-Token Functional Test

The six-thread AProjQ4 generation test completed successfully with demand paging active.

| Metric | Result |
|---|---:|
| Prefill | 0.17 t/s |
| Generation | 0.27 t/s |
| Wall-clock elapsed | 20.89 s |
| Maximum RSS | 6,552,644 KiB |
| Major page faults | 232,065 |
| Filesystem inputs | 52,781,688 |
| Process swaps | 0 |
| Exit status | 0 |

This run establishes functional multi-token execution. It is not directly compared with the earlier eight-thread Q8 generation run because the thread configurations differ.

### Prompt-Specific AProjQ4 Chat Validation

Prompt: `What is the capital of Norway? Reply with only the city name.`

Generated answer: `Oslo`

The captured stdout fragments were interleaved with diagnostic stderr messages and reconstruct to `Oslo`.

| Metric | Result |
|---|---:|
| Prefill | 0.45 t/s |
| Generation | 0.22 t/s |
| Wall-clock elapsed | 61.28 s |
| Maximum RSS | 6,534,168 KiB |
| Major page faults | 304,930 |
| Filesystem inputs | 68,183,920 |
| Process swaps | 0 |
| Exit status | 0 |

### Source Evidence Hashes

| Evidence file | SHA-256 |
|---|---|
| `aprojq4-convert.txt` | `32b532d900a9a74db7f870d4e89ba3c802f3c6efc989e9126344dc431ba9bd82` |
| `aprojq4-inspect.txt` | `ccc71f78ff90582bf6baf8c609324d28387f66faea429eab6435a528bd5e9a1e` |
| `aprojq4-benchmark-6t.txt` | `af1decae4c63d38c868d3cca0e54cd7a8132cc737a49d104bf0534c7251c772f` |
| `aprojq4-decode-profile.txt` | `fcb192c1e1ecf8bc0cbd62677e1bcf2e81eee763fb42369eb00c502e23d45145` |
| `cold-q8.txt` | `249e9722a7b285beb2f494f85454776b111b662fc98eab62222d8de55faed722` |
| `cold-q4.txt` | `f9088a3284d87776d2762a595beb6812de2a7e6da6b1023f73077b4c79964ec6` |
| `norway-chat-aprojq4.txt` | `4291ebf42a42392d4462cf737ee57c49386df5697abdf9a47a8b056119f0c59a` |

### Source-Code Provenance

The foundational AProjQ4 implementation is commit `3b57f9b94d37688a63343475ea6b8402844a1dd2`, titled `Support AProjQ4 GGUFs: Q4_K dense attention projections`.

That change added CPU Q4_K dense-attention execution paths including dense matvec, prequantized decode, batched matmul, and grouped batched matmul support.

### Claim Boundary

This experiment supports the claim that replacing the tested dense attention projections with Q4_K reduced the GGUF file size and substantially reduced cold first-decode storage-fault pressure and latency under the tested configuration. It is a historical controlled comparison, not a repeated current-public-v0.1 performance benchmark, and it does not establish general model-quality equivalence between the Q8 and Q4 quantizations.


## E-003 — V1 Selected-Expert CPU Prefetch

### Classification

- Status: `INCONCLUSIVE`
- Research commit: `0ebc30a6e42965bd513e049d8b269549ed2bec7e`
- Historical tag: `cpu-prefetch-v1-success`
- The historical tag name is retained as provenance and is not treated as the evidence conclusion.
- Cache-drop or zram-reset state was not independently established from the retained experiment evidence.

### Mechanism

V1 added an opt-in `DS4_CPU_EXPERT_PREFETCH` path. For the current token, the router-selected experts were identified and their mmap-backed ranges were submitted to the operating system using `POSIX_MADV_WILLNEED` page-in hints.

The implementation also introduced shared-first scheduling support, but this E-003 comparison isolates selected-expert prefetch only: baseline A versus prefetch B. Shared-first-only and combined configurations are not included in this result.

### Test Parity

The retained A/B command lines used the same AProjQ4 model, prompt `a`, six CPU worker threads, context 32, two generated tokens, temperature zero, demand paging, and decode-detail profiling.

The intended A/B difference was the addition of `DS4_CPU_EXPERT_PREFETCH=1` for B.

### Retained A/B Results

| Pair | Baseline first decode | V1 prefetch first decode | Change | Winner |
|---|---:|---:|---:|---|
| 1 | 4684.435 ms | 4234.063 ms | -9.61% | V1 |
| 2 | 3668.076 ms | 3875.033 ms | +5.64% | Baseline |

Mean first-decode latency changed from 4176.256 ms to 4054.548 ms, approximately -2.91%. V1 won one of the two retained pairs.

### Paging and Process Metrics

| Metric | Pair 1 change | Pair 2 change | Mean A-to-B change |
|---|---:|---:|---:|
| Whole-process elapsed | -4.51% | +3.18% | -0.88% |
| Major page faults | -10.10% | -0.58% | -5.76% |
| Minor page faults | -6.59% | +2.97% | -2.20% |
| Filesystem inputs | -7.32% | +2.36% | -2.90% |
| Maximum RSS | +0.64% | -0.29% | +0.17% |

Process swaps were zero in all four retained runs.

Major page faults decreased in both pairs, but first-decode latency improved in only one pair. The retained evidence therefore does not establish a repeatable latency benefit.

### Output Consistency

The first visible generated output fragment, `package`, matched across the four retained A/B runs.

The retained profiling logs do not provide enough clean generated stdout to establish full captured-output parity. Full output parity is therefore recorded as `NOT ESTABLISHED` rather than inferred.

### Source Evidence Hashes

| Evidence file | SHA-256 |
|---|---|
| `expert-prefetch-bench/r1-A-baseline.txt` | `583d7fb9721a97a667ae0407c10660d52567c9ff811fb49ddcf3f41aa0dbd684` |
| `expert-prefetch-bench/r1-B-prefetch.txt` | `8b64c1c47616d1825643639cc74cf87b7929bd1ba2fa8b36db67fa1a839dd293` |
| `expert-prefetch-bench/r2-A-baseline.txt` | `543fdc02cd8527e95bba85759eaaa36010bb505d325a8d310b274489b8644c67` |
| `expert-prefetch-bench/r2-B-prefetch.txt` | `14185e686144b6877978ab2a367e107a16e6c740aa4e55be43b6b897a74a3777` |

### Claim Boundary

V1 provides evidence that selected-expert WILLNEED hints can alter file-backed paging behavior, including lower major-fault counts in both retained pairs. The retained two-pair experiment does not demonstrate a repeatable first-decode latency improvement, and the cache state before each run cannot be independently reconstructed from the retained evidence.


## E-004 — V2 Select-Once Expert Reuse

### Classification

- Status: `INCONCLUSIVE`
- Direction of retained result: favorable to V2
- Research commit: `13e94562774fa78cb6471c754e2bd252ba82076e`
- Historical tag: `cpu-prefetch-v2-success`
- The historical tag name is provenance, not the evidence conclusion.
- Cache-reset state before individual runs was not independently established from the retained evidence.

### Mechanism

V2 separated expert selection from expert prefetch. Selected expert IDs and router weights were computed once and then reused by the prefetch and routed-expert execution paths.

If preselected values were unavailable, the implementation retained a local-selection fallback.

### Direct V1 vs V2 Design

The retained comparison used the same AProjQ4 model, prompt `a`, six CPU workers, context 32, two generated tokens, temperature zero, demand paging, expert prefetch, shared-first scheduling, token timing, and decode-detail profiling.

The comparison used dedicated V1 and V2 binaries.

### First-Decode Results

| Pair | V1 | V2 | Change | Winner |
|---|---:|---:|---:|---|
| 1 | 3528.267 ms | 3161.060 ms | -10.41% | V2 |
| 2 | 2670.974 ms | 2915.505 ms | +9.16% | V1 |
| 3 | 3062.654 ms | 2918.513 ms | -4.71% | V2 |
| 4 | 2943.967 ms | 2617.349 ms | -11.09% | V2 |

Mean first-decode latency changed from 3051.466 ms to 2903.107 ms, approximately -4.86%.

V2 won three of the four retained pairs.

Mean paired delta: -148.36 ms.

Paired 95% confidence interval: [-592.81, +296.09] ms.

Because the confidence interval crosses zero, the retained first-decode evidence is classified as inconclusive rather than a confirmed performance improvement.

### Paging and Process Metrics

| Metric | V1 mean | V2 mean | Change |
|---|---:|---:|---:|
| Whole-process elapsed | 8.575 s | 8.415 s | -1.87% |
| Major page faults | 80,681.00 | 79,139.75 | -1.91% |
| Minor page faults | 147,845.50 | 139,700.25 | -5.51% |
| Filesystem inputs | 19,254,506 | 18,883,888 | -1.92% |
| Maximum RSS | 6,921,935 KiB | 6,918,688 KiB | -0.05% |

Minor page faults decreased in all four pairs. First-decode latency, major faults, and filesystem input improved in three of four pairs. Process swaps were zero in all eight runs.

### Output Consistency

The first visible generated output fragment, `package`, matched across all eight retained V1/V2 runs.

The profiling logs do not retain enough clean stdout to establish full captured-output parity. Full output parity is therefore recorded as `NOT ESTABLISHED`.

### Source Evidence Hashes

| Evidence file | SHA-256 |
|---|---|
| `v1-v2-direct-ab/r1-v1.txt` | `379f1b48e528bfa02f51da45a60666f3d25e0abb0e0adbaffaf4cd9bdaa4687e` |
| `v1-v2-direct-ab/r1-v2.txt` | `a6e3015d7524d350f93efd8e3d49467edece7ffd37931506f94cc1a579859533` |
| `v1-v2-direct-ab/r2-v1.txt` | `aaa123e2e78d3fee4c5917620197d9c1c4b751d690a2d5441a36dc080eac24f2` |
| `v1-v2-direct-ab/r2-v2.txt` | `563f169d31cc75da5f2155b21b58b2716d1fd880ebca519ae84eed6a869140db` |
| `v1-v2-direct-ab/r3-v1.txt` | `8250cf5be82be6c2662738cc717cb731d508a72b1ddea52390e8bfdc1ad3add4` |
| `v1-v2-direct-ab/r3-v2.txt` | `e11414a380dbc273e2a8dde1790b2ae3a32a94d402eab5edf2703fe804f263b1` |
| `v1-v2-direct-ab/r4-v1.txt` | `83c38f72d25f1d86f245e1008fe20cd75672e2cd10c34b2ac5212ed781cfbfee` |
| `v1-v2-direct-ab/r4-v2.txt` | `2fbb06afa689d836978d86694a1159e6f2238437594821818683eba9284325bb` |

### Claim Boundary

The retained experiment shows a favorable direction for V2: three of four first-decode pairs improved and mean latency fell by 4.86%. The paired confidence interval nevertheless crosses zero, so the evidence does not establish a statistically stable V2 performance improvement. The result supports continued investigation of select-once expert reuse but not a release-level speed claim.


## E-005 — V3 Staged Gate+Up / Down Expert Prefetch

### Classification

- Status: `INCONCLUSIVE`
- Direction of retained result: strongly favorable to V3
- Research commit: `4623b1bf04131d83bffbf2920ef41d5e2c47e77b`
- Historical tag: `cpu-prefetch-v3-success`
- The historical tag is retained as provenance and is not treated as the evidence conclusion.
- Cache-reset state before each individual run was not independently established from the retained evidence.

### Mechanism

V3 added opt-in staged selected-expert prefetch through `DS4_CPU_EXPERT_PREFETCH_STAGED`.

Instead of requesting Gate, Up, and Down expert weights together, the staged path first requests Gate and Up. Down is requested later, immediately before Gate/Up computation, so that Down page-in can overlap useful Gate/Up work.

### Direct V2 vs V3 Design

The retained comparison used the same `./ds4` binary, AProjQ4 model, prompt `a`, six CPU workers, context 32, two generated tokens, temperature zero, demand paging, expert prefetch, shared-first scheduling, token timing, and decode-detail profiling.

For V2, `DS4_CPU_EXPERT_PREFETCH_STAGED` was explicitly unset. For V3, the same variable was set to `1`. This isolates the staged-prefetch feature within the retained command-line configuration.

### First-Decode Results

| Pair | V2 | V3 | Change | Winner |
|---|---:|---:|---:|---|
| 1 | 4306.773 ms | 3103.380 ms | -27.94% | V3 |
| 2 | 3050.669 ms | 2870.727 ms | -5.90% | V3 |
| 3 | 2632.627 ms | 3038.977 ms | +15.44% | V2 |
| 4 | 3552.268 ms | 2612.725 ms | -26.45% | V3 |

Mean first-decode latency changed from 3385.584 ms to 2906.452 ms. The ratio-of-means change is approximately -14.15%.

V3 won three of four retained pairs.

The mean of the four individual percentage changes is -11.21%; this is distinct from the -14.15% ratio-of-means result.

Mean paired delta: -479.132 ms.

Paired 95% confidence interval: [-1644.868, +686.604] ms.

Because the confidence interval crosses zero, the retained result is classified as inconclusive rather than a confirmed performance improvement.

### Paging and Process Metrics

| Metric | V2 mean | V3 mean | Change |
|---|---:|---:|---:|
| Whole-process elapsed | 8.9625 s | 8.4050 s | -6.22% |
| Major page faults | 86,697.00 | 79,183.00 | -8.67% |
| Minor page faults | 157,477.25 | 144,613.50 | -8.17% |
| Filesystem inputs | 20,582,794 | 18,918,122 | -8.09% |
| Maximum RSS | 6,896,064 KiB | 6,925,608 KiB | +0.43% |

Process swaps were zero in all eight retained runs.

Pair 3 moved against the overall direction in first-decode latency, major faults, minor faults, and filesystem input. The aggregate paging reductions must therefore not be represented as four-of-four wins.

### Output Consistency

The first visible generated output fragment, `package`, matched across all eight retained V2/V3 runs.

The profiling logs do not retain enough clean generated stdout to establish full captured-output parity. Full output parity is therefore recorded as `NOT ESTABLISHED`.

### Source Evidence Hashes

| Evidence file | SHA-256 |
|---|---|
| `v2-v3-staged-ab/r1-v2.txt` | `20a4d4fc7ee0fb6b91ce33d0a62363a4f02546f1adcdb3db28223c847247755e` |
| `v2-v3-staged-ab/r1-v3.txt` | `28a77f0bca3db92f04951f29bdba2e2b01eddd808ffdc9e7755052240cea9105` |
| `v2-v3-staged-ab/r2-v2.txt` | `5899b740ea8aea87e6166ac75a4aaf153a1c41b80f7742420192c7d6034d519f` |
| `v2-v3-staged-ab/r2-v3.txt` | `f89d54fb9273442dd9e1e3dccb01bba99115f855ee7ff71aa86d3fd0d222218b` |
| `v2-v3-staged-ab/r3-v2.txt` | `75592dbb62eccd54925802085c7fec2bd27de7df91a8335a7fcd100455b6f0e1` |
| `v2-v3-staged-ab/r3-v3.txt` | `039a61a11b6d0c9254c31319d2da03a7617b201d1dfcbf396f76eca2376c994c` |
| `v2-v3-staged-ab/r4-v2.txt` | `c39985048bfe2c0bf79bdf1165aa86ce51c8c0342645566d7e632d05aa0a3372` |
| `v2-v3-staged-ab/r4-v3.txt` | `5b0abf9a7ae7b69efc42b928d85d85921f81161d23e225c678a6f435762491f3` |

### Claim Boundary

V3 produced a strong favorable direction in the retained experiment: three of four first-decode pairs improved, aggregate first-decode latency fell by 14.15%, and aggregate page-fault and filesystem-input pressure also decreased. However, the substantial run-to-run variation and a paired confidence interval that crosses zero prevent treating the result as a statistically stable performance win. The retained experiment supports staged-prefetch research, not a release-level speed claim.


## E-006 — V4 Expert-Major Prefetch

### Classification

- Status: `INCONCLUSIVE`
- Historical first-decode direction: very strongly favorable to V4
- Historical sustained-decode direction: neutral
- Research commit: `8c66fe4`
- Historical tag: `cpu-prefetch-v4-success`
- The historical tag name is provenance and is not treated as the evidence conclusion.

### Interpretation Boundary

V4 must be evaluated separately for first-decode and sustained behavior. The retained evidence shows a large historical first-decode advantage but does not show that this advantage persists during subsequent decode evaluations.

### E-006A — First-Decode A/B

| Pair | V3 | V4 | Change | Winner |
|---|---:|---:|---:|---|
| 1 | 4430.048 ms | 3011.481 ms | -32.02% | V4 |
| 2 | 3408.926 ms | 2583.157 ms | -24.22% | V4 |
| 3 | 2565.509 ms | 2500.480 ms | -2.53% | V4 |
| 4 | 3117.151 ms | 2597.880 ms | -16.66% | V4 |

Mean first-decode latency changed from 3380.409 ms to 2673.250 ms, a ratio-of-means reduction of approximately 20.92%.

V4 won all four retained first-decode pairs.

Mean paired delta: -707.159 ms.

Paired 95% confidence interval: approximately [-1610.94, +196.62] ms.

Despite the four-of-four directional result, the confidence interval crosses zero because of substantial between-pair variation. The historical result is therefore not treated as a statistically stable general speed claim.

### E-006A — Paging and Process Metrics

| Metric | V3 mean | V4 mean | Change |
|---|---:|---:|---:|
| Whole-process elapsed | 8.9475 s | 8.2675 s | -7.60% |
| Major page faults | 87,878.75 | 76,645.00 | -12.78% |
| Minor page faults | 159,828.50 | 140,774.25 | -11.92% |
| Filesystem inputs | 20,877,808 | 18,298,984 | -12.35% |
| Maximum RSS | 6,916,906 KiB | 6,927,902 KiB | +0.16% |

Process swaps were zero in all eight first-decode runs.

### E-006B — Sustained Decode

For sustained analysis, decode evaluation 1 is kept separate and evaluations 2 through 7 are used as the retained steady-decode window for each run.

| Pair | V3 evals 2-7 mean | V4 evals 2-7 mean | Change | Winner |
|---|---:|---:|---:|---|
| 1 | 3298.988 ms | 3251.342 ms | -1.44% | V4 |
| 2 | 3362.747 ms | 3442.481 ms | +2.37% | V3 |
| 3 | 3522.331 ms | 3502.605 ms | -0.56% | V4 |

Aggregate sustained means were 3394.689 ms for V3 and 3398.809 ms for V4, corresponding to approximately +0.12% for V4.

V4 won two of the three retained sustained pairs.

Mean paired sustained delta: +4.121 ms.

Paired 95% confidence interval: approximately [-162.204, +170.445] ms.

The retained sustained comparison therefore shows no meaningful V4 latency advantage.

### E-006B — Sustained Process Metrics

| Metric | V3 mean | V4 mean | Change |
|---|---:|---:|---:|
| Whole-process elapsed | 28.637 s | 28.533 s | -0.36% |
| Major page faults | 228,774 | 225,389 | -1.48% |
| Minor page faults | 405,358 | 401,382 | -0.98% |
| Filesystem inputs | 55,509,403 | 54,843,984 | -1.20% |
| Maximum RSS | 6,940,855 KiB | 6,946,021 KiB | +0.07% |

Process swaps were zero in all six sustained runs.

The token-level timings exhibit substantial alternating fast and slow decode evaluations in both variants. E-006 records this observation without assigning a reclaim or VM cause; causal attribution is reserved for the later profiling experiments.

### Source Evidence Hashes

#### First-decode A/B

| Evidence file | SHA-256 |
|---|---|
| `v3-v4-expert-major-ab/r1-v3.txt` | `40147a02852af2009d9aecbd1425cd91ac43e950106627273a9a41d152edc095` |
| `v3-v4-expert-major-ab/r1-v4.txt` | `f9a7fabb2bae2dd8ef64ed4b01778a9b55fac483c4a37bd5c76a8566c26dc935` |
| `v3-v4-expert-major-ab/r2-v3.txt` | `3ce589ae6916398529873746afae4e807cee1959ddb9272d3bef015dca69bee1` |
| `v3-v4-expert-major-ab/r2-v4.txt` | `1400a26f4b89525d54f08e6d2e8863b4fd32331b6242aaa5dd4d59499e02d3ac` |
| `v3-v4-expert-major-ab/r3-v3.txt` | `d80da5510f7d5cff06e2aa0a02075ef8875ae99949e7ad3ce90631c4bf419969` |
| `v3-v4-expert-major-ab/r3-v4.txt` | `262c9a9985647c07bc6eadbe7078bc50b19547705c9d11218d4d9a13270f954e` |
| `v3-v4-expert-major-ab/r4-v3.txt` | `3d1f9fd662810d51ae4008138b52b8b854c5b85602237bb4d2d0e57c80cef934` |
| `v3-v4-expert-major-ab/r4-v4.txt` | `f96c5a1afd7b5f69a79199f96513050a7ddd425d5fb692c53d6a2c996ac6e905` |

#### Sustained decode

| Evidence file | SHA-256 |
|---|---|
| `v3-v4-sustained/r1-v3.txt` | `528853bdaaa2c477dc22c3550bf9dc6b56c02410871261e82dd44fc60d8708c2` |
| `v3-v4-sustained/r1-v4.txt` | `fa4d79cabf36c9f00024f133bef88d074253ccbb167c045fa87a8ed1df89a38f` |
| `v3-v4-sustained/r2-v3.txt` | `101e94c222ac0c1ec0ef74295b554a004b51f7b887fe00786b476f9c9d6d8790` |
| `v3-v4-sustained/r2-v4.txt` | `ab60401cb5db960a8aab4a68b6cb59184de9b057e205ac61516053de92c938ca` |
| `v3-v4-sustained/r3-v3.txt` | `6033d7a7a395000609ccad4ea794b391bd111092299f4e4491dda52faaa1e379` |
| `v3-v4-sustained/r3-v4.txt` | `b7ae843131177de75780f4a6c8b37543aaa6fa251d811d33bd6eeecf9792da05` |

### Claim Boundary

The retained historical V4 evidence establishes a strong directional first-decode effect in this experiment, including four-of-four lower first-decode latencies and a 20.92% aggregate reduction. It does not establish a sustained generation-speed improvement: evaluations 2-7 were effectively neutral in aggregate at +0.12%. V4 must therefore not be described as providing a general 20.92% decode-speed improvement.


## E-007 — V5–V13 Profiling Investigation

### Classification

- Status: `DIAGNOSTIC-HISTORICAL`
- Purpose: investigate sustained decode variability and file-backed I/O rather than establish a release performance claim.
- Relevant instrumentation commits include `729cd12` for routed-expert reuse profiling and `2b0af9b` for per-token page-fault and I/O profiling.
- V5 through V13 are retained experiment phases and directories; they do not correspond to nine independent committed releases.

### Investigation Progression

The investigation began with prefetch-placement hypotheses and progressively localized the source of decode I/O.

| Stage | Retained steady-decode result | Evidence conclusion |
|---|---:|---|
| V5 L2 paired | 3578.340 -> 3638.951 ms, +1.69%, L2 wins 1/4 | Rejected as sustained optimization |
| V5 Out-B final | 3703.628 -> 3662.997 ms, -1.10%, Out-B wins 4/6 | Inconclusive |
| V5 Out-B L2 paired | 3721.136 -> 3692.253 ms, -0.78%, Out-B wins 3/4 | Inconclusive |
| V6 intralayer screen | 3605.028 -> 3718.919 ms, +3.16% | No sustained benefit |
| V6 JIT screen | 3485.868 -> 3624.422 ms, +3.97% | No sustained benefit |
| V10 early Gate+Up | 3633.358 -> 3615.799 ms, -0.48%, wins 2/4 | Neutral |
| V11 late Gate+Up | 3721.644 -> 3724.145 ms, +0.07%, wins 2/4 | Neutral |

The V5 L2 paired output files were byte-identical across all eight runs, with common SHA-256 `604196717872bc809ccc148d5fb5f31d12d81378a077aa8fa9c0264fe16d868b`.

### V7 — Phase-Level I/O Localization

Phase instrumentation showed that attention sub-phases generally contributed little or no major-fault I/O in the sampled layers, while `ffn_total` carried hundreds of major faults and substantial block input per layer.

This shifted the investigation from whole-layer timing toward the FFN path.

### V8 — FFN Split

FFN split instrumentation localized the dominant major-fault and block-I/O activity to the routed-expert path. The shared FFN path showed little or no major-fault activity in the sampled layers.

### V9 — Routed MoE Split

The routed path was split further into Gate+Up and Down phases. Both phases showed substantial major faults and block input, while shared Gate+Up and shared Down remained comparatively quiet.

This evidence showed that the sustained I/O problem could not be reduced to only one routed projection.

### V10 / V11 — Prefetch Timing Hypotheses

Moving routed Gate+Up requests earlier or later did not produce a repeatable sustained latency improvement. The retained paired means were effectively neutral.

This weakened the hypothesis that simple prefetch timing alone could remove the observed fast/slow decode behavior.

### V12 — Phase / Token-Position Probe

V12 produced a strong timing difference between the retained P1 and P2 groups, but it is not a valid same-output performance A/B.

P1 output contains the generated `package` continuation while P2 output consists of repeated `a` tokens. The two groups therefore have distinct output hashes and must not be used to claim an 11.23% optimization.

The separate phase-shift probe nevertheless showed substantial token-position variation in major faults and block input, supporting the use of token-level I/O instrumentation.

### V13 — VM-Reclaim Instrumentation Transition

V13 extended the investigation toward token-level VM/reclaim accounting while retaining the routed Gate+Up / Down I/O split.

The retained slice used for E-007 is sufficient to preserve the investigation transition, but E-007 does not claim that V13 alone proves a specific Linux reclaim mechanism.

Stronger evidence relating decode latency to file-backed refault, reclaim, kswapd activity, and VM policy is recorded separately in the later V16 experiment.

### Claim Boundary

E-007 documents how multiple local prefetch-placement hypotheses failed to produce a robust sustained speedup and how profiling localized the dominant file-backed I/O first to FFN, then to routed experts, and finally to both routed Gate+Up and Down. It also records strong token-position-dependent I/O variation. E-007 does not claim that V5-V13 produced a release-ready performance optimization, and it does not use the V12 P1/P2 timing difference as a speed claim because output parity was not preserved.


## E-008 — V14 Shared Gate+Up Residency

### Classification

- Status: `INCONCLUSIVE`
- Direction: favorable
- Research commit: `2b9ff01416ebf6098692d4ac760c87c91584e76e`
- Historical tag: `cpu-v14-shared-gu-causal-success`
- The historical tag name is provenance and is not treated as the public evidence conclusion.

### Mechanism

V14 used `MLOCK_ONFAULT` to request residency for the shared-expert Gate+Up tensors.

The retained B runs report 86 tensors across 86 ranges, representing 731.336 MiB of locked address space under a 979.730 MiB memlock limit.

### Retained Paired Result

The steady window used here is decode evaluations 2 through 19.

| Pair | A baseline | B V14 | Change | Winner |
|---|---:|---:|---:|---|
| 1 | 3490.356 ms | 3460.984 ms | -0.84% | V14 |
| 2 | 3642.879 ms | 3419.927 ms | -6.12% | V14 |
| 3 | 3628.901 ms | 3533.192 ms | -2.64% | V14 |
| 4 | 3747.941 ms | 3553.668 ms | -5.18% | V14 |

Mean steady-decode latency changed from 3627.519 ms to 3491.943 ms, approximately -3.74%.

V14 won all four retained steady-decode pairs.

Mean paired delta: -135.577 ms.

Paired 95% confidence interval: approximately [-277.75, +6.59] ms.

Because the confidence interval crosses zero, the retained result is classified as inconclusive despite its consistent favorable direction.

### First-Decode Context

Mean first-decode latency changed from 2836.152 ms to 2798.855 ms, approximately -1.32%, with V14 lower in three of four pairs.

### Output Consistency

All eight retained output files are byte-identical with SHA-256 `b6cf063007cd61da094a9bf5000df2ff8ef74ba74efe658a16fd6d2de986d81f`.

### Source Evidence Hashes

| Evidence file | SHA-256 |
|---|---|
| `v14-shared-gu-ab-final/1-A.log` | `7228b9de81254234ffe65243e1b5726022486d2b0dd10f50ff3b7dd4e5c2630b` |
| `v14-shared-gu-ab-final/1-B.log` | `ea4c3ba9ee8a5b45d62946fc288a9df79bb400955ca270a5e9aeb3c031a1fec6` |
| `v14-shared-gu-ab-final/2-A.log` | `4658712c8f8851884c56dcb8610a48bc501792b96e2e01b91226ca39077f6f94` |
| `v14-shared-gu-ab-final/2-B.log` | `376bb9f118cb6234d1dba0c99fe2cde4fe6f91218d0e6b9c35ed349a91525c96` |
| `v14-shared-gu-ab-final/3-A.log` | `f7a2e1021e175ef5424e75add6e4e58367aa105f6cb7c293f63c4deba291a3c4` |
| `v14-shared-gu-ab-final/3-B.log` | `b6e869d94340befb01878fe38a9e96dc32b8dc290bb2cec3e91604f87b559fa3` |
| `v14-shared-gu-ab-final/4-A.log` | `55f1a20c0f124da0767cdb60cf9ce9170d4910cb6c6c566f75ab5addb332c106` |
| `v14-shared-gu-ab-final/4-B.log` | `cc279d8abe55c57f73638c616bd8ec021c916c25a02ba2a749ced8812c753911` |

### Claim Boundary

V14 provides a consistent favorable historical signal for shared Gate+Up residency: all four retained steady-decode pairs improved and the aggregate reduction was 3.74%. The small sample and paired confidence interval crossing zero prevent treating this as a confirmed release-level speed improvement. The result supports the shared-residency hypothesis but does not establish a universal benefit.


## E-009 — V15 Shared Down Residency

### Classification

- Status: `INCONCLUSIVE`
- Evidence level: single-pair screening experiment
- Sustained direction: neutral to slightly unfavorable
- Research commit: `da251ac093006ad82599a47d0dab4fa16c55f3ad`
- Historical tag: `cpu-v15-shared-down-causal-success`
- The historical tag name is provenance and is not treated as the public evidence conclusion.

### Mechanism

V15 used `MLOCK_ONFAULT` to request residency for shared-expert Down tensors.

The retained B run reports 43 tensors across 43 ranges, representing 365.668 MiB of locked address space under a 979.730 MiB memlock limit.

### Screening Result

The retained screen contains one baseline/V15 pair.

First-decode latency changed from 3020.096 ms to 2706.578 ms, approximately -10.38%.

For decode evaluations 2 through 19, mean latency changed from 3560.152 ms to 3595.627 ms, approximately +1.00%.

The first evaluation therefore improved while the sustained window was slightly slower.

With only one retained pair, no paired confidence interval or repeatability claim is appropriate.

### Output Consistency

The two retained output files are byte-identical with SHA-256 `b6cf063007cd61da094a9bf5000df2ff8ef74ba74efe658a16fd6d2de986d81f`.

### Source Evidence Hashes

| Evidence file | SHA-256 |
|---|---|
| `v15-shared-down-ab-screen/a.log` | `5b79325ed1bbe940b4795f2153572b9986ad96e5cfc93529d3b1ca925de06a39` |
| `v15-shared-down-ab-screen/b.log` | `a6f06b1bd3d27d46e9616e626723a02217c8b5c8aae47a998b48e5291768e52e` |

### Claim Boundary

V15 shows that locking shared Down changes first-decode behavior, but the retained single-pair screen does not demonstrate a sustained performance benefit. The steady-decode mean was approximately 1.00% slower with V15 enabled. This experiment is retained as a diagnostic result and must not be represented as a confirmed optimization.


## E-011 — V4 Latest-Upstream Reproduction

### Classification

- Status: `INCONCLUSIVE`
- Release decision: rejected for v0.1 default
- Purpose: test whether the large historical V4 first-decode effect reproduced on the current-upstream public candidate.

### Controlled Cold Design

Four balanced A/B pairs were run with the same public-candidate binary and numerical workload. A disabled V4 expert-major prefetch and B enabled it.

Before each controlled run, zram was reset and filesystem caches were dropped. The tested system used swappiness 10 and approximately 6.5 GiB available memory before the run.

All eight numerical result files were identical, so numerical-output parity was verified.

### Results

| Pair | A: V4 off | B: V4 on | Change | Winner |
|---|---:|---:|---:|---|
| 1 | 5.550 s | 4.900 s | -11.71% | V4 |
| 2 | 4.860 s | 4.850 s | -0.21% | V4 |
| 3 | 4.910 s | 5.070 s | +3.26% | Baseline |
| 4 | 4.860 s | 4.880 s | +0.41% | Baseline |

Mean elapsed time changed from 5.045 s to 4.925 s, approximately -2.38%.

V4 won two of four pairs.

Mean paired delta: -0.120 s.

Paired 95% confidence interval: [-0.694, +0.454] s.

Other pooled metrics were effectively unchanged: maximum RSS +0.00%, major faults -0.02%, filesystem inputs -0.01%, and swaps remained zero.

### Output Consistency

All eight retained numerical result files matched. Their common SHA-256 was `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3`.

### Local Source Evidence Hashes

The raw controlled-run files contain machine-local metadata and are not copied into the public repository. Their hashes are retained here for traceability.

| Local evidence file | SHA-256 |
|---|---|
| `pair1-A-OFF.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair1-A-OFF.time` | `815ceb8f9ad1a8c5427ada9a7d20da5d0d8f62c2778a1e157409c6e75bfd8f91` |
| `pair1-B-V4.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair1-B-V4.time` | `3a14f6013c75158615740a4ece8877172a6666e62a989f32e9fe03fe5a518166` |
| `pair2-A-OFF.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair2-A-OFF.time` | `c45f54d7c7de37bac66465d734e8c4c0fde4a4fa9775fd70a59e5975f4b1ced2` |
| `pair2-B-V4.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair2-B-V4.time` | `230fac52cbe0cd442e3f71de0fcfc66424386782460093579d4f84910ca74da4` |
| `pair3-A-OFF.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair3-A-OFF.time` | `3a752a8c5d154665076718fb2622928dacf47407aa625aa31b5c2f7a4238f8e9` |
| `pair3-B-V4.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair3-B-V4.time` | `0e27a1912bc5729d99bdd7c218ee3bc0ba6810293c0e5be3f2445308e5c62819` |
| `pair4-A-OFF.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair4-A-OFF.time` | `546b750d5826400b86d92957898d164bc51df8fc6deb9c7805a9901c1d8a6548` |
| `pair4-B-V4.result` | `a2d1012ebdea42dfaf67e0af5c2795bb32c74c9bc184e8c4f0d9e5d123d9ddb3` |
| `pair4-B-V4.time` | `a894571aa8877f1dae98a9e9fe649738e3199c0f2d8ece4ae6642a37ce4e1db1` |

### Claim Boundary

The latest-upstream reproduction does not support carrying the historical V4 speed claim into v0.1. The observed mean direction was favorable by 2.38%, but only two of four pairs improved and the paired confidence interval crossed zero. V4 expert-major prefetch is therefore excluded from the public v0.1 default.


## E-012 — Public v0.1 Controlled Cold First-Token Validation

### Classification

- Status: `VERIFIED-PUBLIC`
- Public candidate: `e330748f66cf2024c6310e37cb4885d38d58054f`
- AProjQ4 CPU port: `155fe2ee7d39f4bf91045f0b97bcd93f5ace4d50`
- Upstream base: `84cc882352757baf628a1776badf7cc54d584e28`
- Backend: CPU only

### Controlled Cold State

The controlled public-candidate validation reset zram, dropped filesystem caches, and used Linux swappiness 10 before the run.

The host had 7.7 GiB physical RAM and approximately 6.5 GiB available memory in the recorded cold state.

The AProjQ4 GGUF remained mmap-backed and CPU whole-model prefetch was disabled through the opt-in demand-paging path.

### Result

| Metric | Public cold validation |
|---|---:|
| Whole-process elapsed | 5.33 s |
| Maximum RSS | 6,204,632 KiB |
| Major page faults | 55,553 |
| Minor page faults | 96,819 |
| Filesystem inputs | 12,787,656 |
| Process swaps | 0 |
| System pgmajfault delta | +4,637 |
| zram after run | 0 B |

The run completed successfully on the current public candidate and exercised real file-backed model I/O.

### Interpretation

This establishes that the current v0.1 candidate can execute an exact CPU whole-model first-token diagnostic for the 78.62 GiB AProjQ4 DeepSeek V4 Flash GGUF on the tested 7.7 GiB RAM machine using demand paging.

It does not mean that the model fits in RAM. The model remains much larger than physical memory and pages are faulted from NVMe as required.

### Local Source Evidence Hashes

The raw controlled-run files remain outside the public repository because they contain machine-local execution metadata. Their hashes are recorded for traceability.

| Local evidence file | SHA-256 |
|---|---|
| `ds4-public-step5b3.log` | `535e9eddd9f3e31cdfc30648b8af563ee617a0b78dfcdedef75352bbde197351` |
| `ds4-public-step5b3.time` | `abe9ba6aded6e5ca058ae35d6cd59811dcbb33d1ff69f69cae5cae9f52601285` |
| `ds4-public-step5b3.vm.before` | `6927ba4b1b4ad720ded15f87c6039e610a72865f947a612556436898e356bfd6` |
| `ds4-public-step5b3.vm.after` | `54d39d7c2c34d270d71231629ec39bab3ebd354c4e9910caf22856e00c616661` |

### Claim Boundary

The 5.33-second result is a controlled cold first-token diagnostic, not a production generation-speed benchmark. E-012 supports CPU-only functional execution under severe RAM-to-model-size mismatch and validates the public demand-paging path; it does not establish sustained generation throughput or universal performance on other storage, kernels, or hardware.

## E-013 — V16.2 Swappiness 100 vs 150

### Classification

- Status: `INCONCLUSIVE`
- Scope: historical Linux VM follow-up experiment
- Backend: CPU only
- Comparison: `vm.swappiness=100` vs `vm.swappiness=150`
- Balanced paired runs: 4
- Sustained measurement window: decode evaluations 2 through 19
- Output parity: identical across all eight runs
- Public-release relationship: follow-up historical evidence; not a current v0.1 performance claim

### Result

| Metric | Swappiness 100 | Swappiness 150 | Change |
| --- | ---: | ---: | ---: |
| Mean steady decode | 3351.506 ms | 3297.958 ms | -1.60% |
| Pair wins | 1/4 | 3/4 | — |
| Major faults | 23,767.72 | 22,433.46 | -5.61% |
| Filesystem input | 5,189,073.67 | 4,822,966.44 | -7.06% |
| File refaults | 507,423.83 | 460,170.10 | -9.31% |
| kswapd scan | 1,687,742.67 | 1,623,640.28 | -3.80% |
| kswapd steal | 660,960.03 | 617,370.94 | -6.59% |
| Swap-in | 6,218.28 | 8,449.49 | +35.88% |
| Swap-out | 10,063.54 | 12,696.74 | +26.17% |
| ZRAM growth | 658.42 MiB | 682.23 MiB | +3.62% |
| Memory PSI some | 12.030 s | 10.968 s | -8.83% |
| Memory PSI full | 11.942 s | 10.869 s | -8.99% |

Paired steady-decode mean difference:

`-53.548 ms`

Relative change:

`-1.60%`

B150 wins:

`3/4`

Paired 95% confidence interval:

`[-196.995, +89.898] ms`

The confidence interval crosses zero, so this experiment does not establish a reliable sustained-decode advantage for swappiness 150 over 100.

### Interpretation

The direction was favorable to swappiness 150 and was accompanied by fewer major faults, less filesystem input, fewer file refaults, and lower memory PSI.

However, swap activity increased and the paired confidence interval crossed zero. The result is therefore classified as inconclusive rather than as a performance improvement.

Combined with E-010, the result is consistent with diminishing returns after the large improvement observed between swappiness 10 and 100.

### Methodology Boundary

The V16.2 reset procedure reset zram and dropped filesystem caches before each run, but did not fully clear the disk-backed swapfile between runs.

This limits the experiment to historical research evidence. It should not be interpreted as a universal Linux tuning recommendation.

Sanitized numeric source:

`benchmarks/v16.2-swappiness-100-vs-150.csv`

## E-014 — V16.3 Swappiness 100 vs 200

### Classification

- Status: `STRONG-HISTORICAL`
- Scope: historical Linux VM follow-up experiment
- Backend: CPU only
- Comparison: `vm.swappiness=100` vs `vm.swappiness=200`
- Balanced paired runs: 4
- Sustained measurement window: decode evaluations 2 through 19
- Output parity: identical across all eight runs
- Public-release relationship: historical evidence only; not a current v0.1 performance claim

### Result

| Pair | Swappiness 100 | Swappiness 200 | Change |
| --- | ---: | ---: | ---: |
| 1 | 3506.335 ms | 5888.745 ms | +67.95% |
| 2 | 3346.007 ms | 5574.943 ms | +66.61% |
| 3 | 3282.993 ms | 5735.438 ms | +74.70% |
| 4 | 3314.906 ms | 5881.953 ms | +77.44% |
| Mean | 3362.560 ms | 5770.270 ms | +71.60% |

A100 wins:

`4/4`

B200 wins:

`0/4`

Paired mean difference:

`+2407.710 ms`

Paired 95% confidence interval:

`[+2182.696, +2632.723] ms`

The confidence interval excludes zero and every balanced pair regressed at swappiness 200.

### First-Decode Observation

Mean first decode:

- swappiness 100: `3975.231 ms`
- swappiness 200: `3128.571 ms`
- apparent change: approximately `-21.30%`
- B200 wins: `4/4`
- paired 95% confidence interval: `[-2113.612, +420.293] ms`

The first-decode confidence interval crosses zero. This apparent advantage is therefore not treated as a reliable performance result.

More importantly, it does not compensate for the large sustained-decode regression.

### Interpretation

Swappiness 200 was decisively harmful to sustained decode on the tested machine and workload.

The regression was large, consistent across all four balanced pairs, and statistically separated from zero in the paired analysis.

Memory pressure also became very high during the B200 runs, consistent with excessive reclaim and swap activity under this workload.

E-014 therefore rejects swappiness 200 as a useful setting for this tested configuration.

Combined V16 evidence now shows:

- swappiness 10 to 100: strong historical improvement
- swappiness 100 to 150: small favorable but inconclusive trend
- swappiness 100 to 200: strong sustained-decode regression

The current candidate region for further research is therefore approximately swappiness 100 to 150 on this tested system and workload.

### Methodology Boundary

As in V16.2, zram was reset and filesystem caches were dropped, but the disk-backed swapfile was not fully reset between runs.

That limitation does not erase the large and consistent V16.3 regression, but future VM experiments use a stricter full swap-tier reset before drawing finer-grained tuning conclusions.

Sanitized numeric source:

`benchmarks/v16.3-swappiness-100-vs-200.csv`

### Claim Boundary

E-014 is historical system-tuning evidence. It does not establish an optimal Linux swappiness value for other machines, kernels, storage devices, memory sizes, model formats, or workloads.

## E-015 — V16.4 Swappiness 150 vs 175

### Classification

- Status: `INCONCLUSIVE`
- Scope: historical Linux VM follow-up experiment
- Backend: CPU only
- Comparison: `vm.swappiness=150` vs `vm.swappiness=175`
- Balanced paired runs: 4
- Sustained measurement window: decode evaluations 2 through 19
- Output parity: identical across all eight runs
- Reset methodology: full reset of both disk-backed swap and zram before every run
- Public-release relationship: historical follow-up evidence; not a current v0.1 performance claim

### Controlled Reset

V16.4 strengthened the VM methodology used in earlier V16 experiments.

Before every run:

- zram swap was disabled,
- the disk-backed swapfile was disabled,
- all swapped pages were returned to RAM,
- the disk swapfile was re-enabled empty,
- zram was re-enabled empty,
- filesystem caches were dropped,
- and only then was the test swappiness value applied.

Recorded pre-run swap usage was `0 B` for both swap tiers in all eight runs.

### Steady-Decode Result

| Pair | Swappiness 150 | Swappiness 175 | Change |
| --- | ---: | ---: | ---: |
| 1 | 3227.757 ms | 3193.284 ms | -1.07% |
| 2 | 3399.986 ms | 3222.583 ms | -5.22% |
| 3 | 3350.355 ms | 3336.179 ms | -0.42% |
| 4 | 3353.080 ms | 3293.923 ms | -1.76% |
| Mean | 3332.794 ms | 3261.492 ms | -2.14% |

B175 wins:

`4/4`

Paired mean difference:

`-71.302 ms`

Paired 95% confidence interval:

`[-187.598, +44.995] ms`

The direction favored swappiness 175 in every pair, but the paired confidence interval crossed zero.

The result therefore remains `INCONCLUSIVE`.

### VM / Memory-Pressure Result

Mean counter changes for swappiness 175 relative to 150:

| Metric | Change |
| --- | ---: |
| Major faults | +12.41% |
| File refaults | -11.91% |
| kswapd scan | -3.10% |
| kswapd steal | -9.04% |
| Direct scan | +7.57% |
| Direct steal | +13.19% |
| Swap-in | +19.41% |
| Swap-out | +9.65% |
| Memory PSI some | -12.52% |
| Memory PSI full | -12.46% |
| ZRAM growth | +0.70% |

The VM signal is mixed.

Swappiness 175 reduced file refaults, kswapd activity, and measured memory PSI, while increasing major faults, direct reclaim, and swap traffic.

This is consistent with a redistribution of memory pressure rather than a universal reduction in VM work.

### Interpretation

Swappiness 175 produced lower sustained-decode latency in all four balanced pairs under the strengthened full-reset methodology.

However, the paired 95% confidence interval crossed zero, so V16.4 does not establish a reliable speed improvement.

The result supports a consistent favorable trend toward 175 on this tested system and workload.

Combined V16 evidence now indicates:

- swappiness 10 to 100: strong historical improvement
- swappiness 100 to 150: favorable but inconclusive
- swappiness 150 to 175: favorable in 4/4 pairs but inconclusive
- swappiness 100 to 200: strong sustained-decode regression

The degradation boundary is therefore bracketed between swappiness 175 and 200 for the tested workload.

This does not establish swappiness 175 as an optimum.

### Sanitized Numeric Source

`benchmarks/v16.4-swappiness-150-vs-175.csv`

### Local Evidence Hash

The local research summary is retained outside the public repository.

`e1d3bcf7b91c56d9d1c59731cb3eff9112fd48b8aea188f5c354561b8dc7b6d8`

### Claim Boundary

E-015 is historical system-tuning evidence.

It does not establish an optimal Linux swappiness value for other machines, kernels, RAM capacities, storage devices, swap configurations, model formats, or workloads.

## E-016 — V16.5 Swappiness 175 vs 188

### Classification

- Status: `INCONCLUSIVE`
- Scope: historical Linux VM follow-up experiment
- Backend: CPU only
- Comparison: `vm.swappiness=175` vs `vm.swappiness=188`
- Balanced paired runs: 4
- Sustained measurement window: decode evaluations 2 through 19
- Output parity: identical across all eight runs
- Reset methodology: full disk-swap and zram reset before every run
- Public-release relationship: historical follow-up evidence; not a current v0.1 performance claim

### Controlled Reset

V16.5 retained the strengthened full swap-tier reset introduced in V16.4.

The disk-backed swapfile started at `0 B` in all eight runs.

Seven runs also started with zram at `0 MiB`.

Pair 4 A175 recorded approximately `0.250 MiB` of zram use before the workload. This remained well inside the predefined reset tolerance and is retained here as a methodology detail rather than hidden.

### Steady-Decode Result

| Pair | Swappiness 175 | Swappiness 188 | Change |
| --- | ---: | ---: | ---: |
| 1 | 3067.455 ms | 3235.679 ms | +5.48% |
| 2 | 3213.748 ms | 3267.598 ms | +1.68% |
| 3 | 3250.521 ms | 3306.234 ms | +1.71% |
| 4 | 3360.558 ms | 3371.811 ms | +0.33% |
| Mean | 3223.071 ms | 3295.331 ms | +2.24% |

A175 wins:

`4/4`

B188 wins:

`0/4`

Paired mean difference:

`+72.260 ms`

Paired 95% confidence interval:

`[-34.655, +179.175] ms`

Every pair favored swappiness 175, but the paired confidence interval crossed zero.

The sustained result therefore remains `INCONCLUSIVE`.

### VM / Memory-Pressure Result

Mean changes for swappiness 188 relative to 175:

| Metric | Change |
| --- | ---: |
| Major faults | +22.95% |
| File refaults | +1.16% |
| kswapd scan | -1.85% |
| kswapd steal | +1.43% |
| Direct scan | -8.06% |
| Direct steal | -5.48% |
| Swap-in | +29.66% |
| Swap-out | +15.70% |
| Memory PSI some | +2.82% |
| Memory PSI full | +2.79% |
| ZRAM growth | -0.43% |

The VM signal is mixed.

Swappiness 188 produced substantially more major faults and swap traffic, together with slightly higher memory PSI, while some direct reclaim counters decreased.

These observations support a change in reclaim behavior but do not establish a single causal mechanism for the decode-latency difference.

### Interpretation

Swappiness 188 produced slower steady-decode latency than 175 in all four balanced pairs.

However, the paired 95% confidence interval crossed zero, so V16.5 does not establish a statistically reliable regression at 188.

Combined with V16.4:

- swappiness 150 to 175 favored 175 in 4/4 pairs
- swappiness 175 to 188 favored 175 in 4/4 pairs
- both comparisons remained statistically inconclusive
- swappiness 200 had previously produced a strong sustained-decode regression

The current candidate turnover region is therefore approximately swappiness 175 to 188 for this tested workload.

Swappiness 175 is the strongest candidate point observed so far, but it is not established as an optimum.

### Sanitized Numeric Source

`benchmarks/v16.5-swappiness-175-vs-188.csv`

### Local Evidence Hash

The local research summary remains outside the public repository.

`83dd8f5d548781c2e25ec1fdb755a3bd3047ff6ad22b8114eaae320f429518e2`

### Claim Boundary

E-016 is historical system-tuning evidence.

It does not establish an optimal Linux swappiness value for other hardware, kernels, storage devices, memory capacities, swap configurations, model formats, or workloads.

## E-017 — V16.6 Swappiness 175 vs 182

### Classification

- Status: `INCONCLUSIVE`
- Interpretation: effectively neutral between swappiness 175 and 182
- Scope: historical Linux VM follow-up experiment
- Backend: CPU only
- Comparison: `vm.swappiness=175` vs `vm.swappiness=182`
- Balanced paired runs: 4
- Sustained measurement window: decode evaluations 2 through 19
- Output parity: identical across all eight runs
- Reset methodology: full disk-swap and zram reset procedure before every run
- Public-release relationship: historical follow-up evidence; not a current v0.1 performance claim

### Controlled Reset

V16.6 retained the strengthened swap-reset methodology used in V16.4 and V16.5.

Pre-run residual swap usage remained extremely small:

| Run | Disk swap | ZRAM |
| --- | ---: | ---: |
| 1-A175 | 0.000 MiB | 0.000 MiB |
| 1-B182 | 0.000 MiB | 0.250 MiB |
| 2-A175 | 0.000 MiB | 0.000 MiB |
| 2-B182 | 0.000 MiB | 0.000 MiB |
| 3-A175 | 0.000 MiB | 0.000 MiB |
| 3-B182 | 0.250 MiB | 0.000 MiB |
| 4-A175 | 0.000 MiB | 0.000 MiB |
| 4-B182 | 0.250 MiB | 0.000 MiB |

All residual values were far below the predefined 16 MiB reset tolerance.

### Steady-Decode Result

| Pair | Swappiness 175 | Swappiness 182 | Change |
| --- | ---: | ---: | ---: |
| 1 | 3105.582 ms | 3216.106 ms | +3.56% |
| 2 | 3251.928 ms | 3163.885 ms | -2.71% |
| 3 | 3255.752 ms | 3340.080 ms | +2.59% |
| 4 | 3446.469 ms | 3403.626 ms | -1.24% |
| Mean | 3264.933 ms | 3280.924 ms | +0.49% |

A175 wins:

`2/4`

B182 wins:

`2/4`

Paired mean difference:

`+15.992 ms`

Paired 95% confidence interval:

`[-137.436, +169.419] ms`

The pair split was even, the mean difference was small, and the paired confidence interval crossed zero by a wide margin.

The sustained result therefore remains `INCONCLUSIVE`.

### VM / Memory-Pressure Result

Mean changes for swappiness 182 relative to 175:

| Metric | Change |
| --- | ---: |
| Major faults | +10.25% |
| File refaults | -3.47% |
| kswapd scan | -1.32% |
| kswapd steal | -2.04% |
| Direct scan | +1.06% |
| Direct steal | -7.81% |
| Swap-in | +11.95% |
| Swap-out | +6.37% |
| Memory PSI some | -1.99% |
| Memory PSI full | -1.98% |
| ZRAM growth | +0.36% |

The VM signal is mixed.

Swappiness 182 increased major faults and swap traffic while reducing file refaults, some reclaim activity, and measured memory PSI.

No single VM mechanism reliably distinguishes 182 from 175 in this experiment.

### Interpretation

Swappiness 175 and 182 produced effectively equivalent sustained-decode performance in this four-pair comparison.

The pair split was 2/4 versus 2/4 and mean steady decode differed by only 0.49%.

There is no reliable evidence from V16.6 that swappiness 182 is faster or slower than 175.

Under this tested configuration, the range from approximately 175 through 182 currently behaves like a performance plateau rather than a clearly ordered optimum.

Combined with the neighboring experiments:

- swappiness 150 to 175 favored 175 in 4/4 pairs but remained inconclusive
- swappiness 175 to 182 was effectively neutral
- swappiness 175 to 188 favored 175 in 4/4 pairs but remained inconclusive
- swappiness 200 previously produced a strong sustained-decode regression

The current degradation-onset bracket is therefore approximately swappiness 182 to 188 for this tested workload.

### Sanitized Numeric Source

`benchmarks/v16.6-swappiness-175-vs-182.csv`

### Local Evidence Hash

The local research summary remains outside the public repository.

`d94a68305e838fa13fea9c6587ac1bd313f9e09afc5881cf4889749ea7f3cc67`

### Claim Boundary

E-017 is historical system-tuning evidence.

It does not establish an optimal Linux swappiness value or a universal plateau for other hardware, kernels, storage devices, memory capacities, swap configurations, model formats, or workloads.
