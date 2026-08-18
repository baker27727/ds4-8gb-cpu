# V33 Clean R1 Cache — Controlled Wall-Time Regression Evidence

## Purpose

This document records a controlled negative result for the experimental V33 R1 expert-weight cache. It is research evidence only; it is not a production optimization and does not modify the protected runtime winner.

## Protocol and scope

The canonical 60-token workload was run with cache OFF versus cache ON/reclaimable using 8 predefined balanced AB/BA pairs. A and B denote those two cache states; pair order was balanced to reduce ordering effects. There were 16/16 valid runs and 8/8 complete pairs. No warm-up, replacement runs, cache manipulation, or mechanism-causality inference was used.

Exact output parity: **PASS**. The paired statistical unit is one complete pair, n=8.

## Results

| Measure | Result |
|---|---:|
| Mean B−A | **+60.156027561 s** |
| 95% paired t CI | **[+56.244857128 s, +64.067197994 s]** |
| Sample SD | **4.678316952 s** |
| Mean paired speedup | **−49.682685837%** |
| Median paired speedup | **−49.877608447%** |

## Classification and interpretation

**Classification: REGRESSION**

V33 cache ON/reclaimable was reliably slower than cache OFF for this exact host, workload, and protocol. Mechanism functionality does not imply performance benefit. This result rejects V33 promotion in its tested form. It does not explain the causal mechanism and does not generalize to other hosts, token counts, or workloads.

The paired observations are published in [`benchmarks/v33-r1-cache-60t-paired.csv`](../../benchmarks/v33-r1-cache-60t-paired.csv).
