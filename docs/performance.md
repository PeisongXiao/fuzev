# ExLlamaV3 Short-Workload Performance

This benchmark times three short modes:

- **Generation:** a short Python-file chat prompt.
- **Decision:** a two-option Choice between a source file and a cooking recipe.
- **Mixed:** generation first, then the decision on its retained prefix.

To serve generation and decisions, see the [installation guide](install.md).

## Setup

- **Platform:** NVIDIA GB10 (DGX Spark), ARM64, Python 3.12,
  PyTorch 2.13.0+cu130 (CUDA 13.0).
- **Build:** a pre-release standalone ExLlamaV3 server and engine from the
  archived research evaluation servers, with
  [the prefill optimizations from exllamav3-fuzev (`exllamav3.optimized`)](https://github.com/PeisongXiao/exllamav3-fuzev/blob/main/doc/fuzev-fork.md#prefill-optimizations)
  enabled.
- **Model settings:** MTP off, FP16 KV.
  [Models](models.md#recorded-benchmark-settings) lists checkpoint revisions and
  embedding storage.
- **Generation:** recorded chat template with thinking disabled; temperature 1,
  top-p 0.95, top-k 20, seed 17; 32 requested output tokens. The JSON keeps
  observed prompt and output counts, including early EOS.
- **Runs:** one warmup and three measured runs per mode and stream count.
- **Prefix reuse:** unique leading prefixes prevent reuse between pipelines.
  Mixed mode reuses the accepted generation prefix within a pipeline.
- **Concurrency:** the measured server ran one request at a time, so four-stream
  times include queueing. The released integration submits requests to the
  ExLlamaV3 Generator, which batches concurrent jobs; its four-stream behavior
  is unmeasured.

## Measurements

Wall times are mean per-pipeline seconds, including queueing. Brackets give the
minimum and maximum of the three run means. Pipeline time excludes prefix
release; the JSON's separate run-wall measurements include it.

| Configuration | Streams | Gen Prefill tok/s | Gen Decode tok/s | Gen Wall s [Range] | Decision Wall s [Range] | Mixed Wall s [Range] |
| --- | ---: | ---: | ---: | --- | --- | --- |
| ExLlamaV3 EXL3 3.05 bpw | 1 | 936.17 | 24.19 | 1.77 [1.75, 1.79] | 0.80 [0.80, 0.81] | 2.48 [2.13, 3.16] |
| ExLlamaV3 EXL3 3.05 bpw | 4 | 930.84 | 23.89 | 4.47 [4.43, 4.51] | 2.01 [2.01, 2.01] | 8.11 [8.07, 8.16] |
| ExLlamaV3 EXL3 4.05 bpw | 1 | 944.25 | 23.20 | 1.82 [1.81, 1.83] | 0.81 [0.81, 0.82] | 2.24 [2.20, 2.28] |
| ExLlamaV3 EXL3 4.05 bpw | 4 | 939.71 | 23.31 | 4.54 [4.52, 4.55] | 2.06 [2.05, 2.07] | 8.26 [8.19, 8.31] |
| ExLlamaV3 EXL3 5.05 bpw | 1 | 865.65 | 21.23 | 1.99 [1.96, 2.01] | 0.91 [0.90, 0.91] | 2.75 [2.42, 3.40] |
| ExLlamaV3 EXL3 5.05 bpw | 4 | 860.95 | 20.86 | 5.03 [5.01, 5.05] | 2.26 [2.25, 2.26] | 9.10 [9.06, 9.14] |

Engine prefill includes the first sampled token; decode excludes it. These
phase timers measure engine intervals.

- [Aggregate data and measured runs](performance-results.json): all timing
  samples and observed token counts.
- [Source timing records](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/results/README.md#short-performance):
  the individual measured runs.

## Decision Latency

The [API limitations](api.md#limitations) cover decision cost, cache-block
reuse and the MTP trade-off.

### ExLlamaV3 Decision Diagnostics

These timings cover the installed ExLlamaV3 EXL3 4.05 bpw path on GB10 with an
8-bit KV cache, CPU head, the default prefill optimizations, 8,192-token prefill
chunks and MTP off.

For SystemOne requests with two or more questions and a state longer than one
256-token cache page, the integration prefills the shared state once before the
questions run.

- **One request with three questions over 2,048 context words:** 8.74 s with the state prefilled
  separately for each question; 5.32 s with it prefilled once.
- **One 64-word Choice with eight options:** 3.64 s, with a 223-token query and
  240 tokens per option.

The [archived record](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/configs/smoke-native-exl3.json)
keeps the timings, answers and token counts.

Decision probabilities can differ slightly:

- between a request's first run and later repeats, once its prefix pages are cached;
- between server starts, because Triton selects kernel configurations by timing
  when it first compiles them.

## Limitations

- Published serving builds have no end-to-end quality or performance benchmark.
- This benchmark compares quantizations on the recorded builds.
- Three short runs give observed spread, with no confidence interval. Small
  timing differences need repeated measurements to establish reproducible speedups.
- The decision diagnostics are single timings. The shared-state comparison is one
  ordered pair with no separate warmup, so first-use kernel compilation can add
  to the first request's time.
- The 64-word Choice fits within one 256-token cache page, so it does not time
  contexts that cross a page boundary.
- The short-workload table uses a pre-release fork build. Its extension predates
  the released CUDA-graph kernel-argument fix, and its optimized prefill casts
  gated-delta-network operands to FP16. These measurements were not repeated on
  the released build ([fork limitations](https://github.com/PeisongXiao/exllamav3-fuzev/blob/main/doc/fuzev-fork.md#limitations)).

See also:

- [ExLlamaV3 generation control](integration.md#exllamav3-generation-control): upstream and fork comparison.
- [Benchmarks](benchmarks.md): decision quality.
- [Integration](integration.md): long-context and MTP checks.
- [API limitations](api.md#limitations): supported engine settings and images.
