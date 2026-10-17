# Generation And Decision Integration

FuZev attaches a shared decision head to an ExLlamaV3 Generator.
The engine runs scheduling and caches; each decision scores the head afresh.

To serve, see the [installation guide](install.md), [API reference](api.md) and
[ExLlamaV3 integration](https://github.com/PeisongXiao/fuzev-exllamav3-plugin/tree/v1.1).
GB10 source-installation checks are in
[verified source installations](install.md#verified-source-installations).

## Recorded Measurement Setup

All checks:

- **Platform:** NVIDIA GB10 (DGX Spark), ARM64.
- **Builds:** pre-release builds of the v1.1 engine fork, runtime and
  integration plugin.

ExLlamaV3 checks:

- **Model:** [turboderp EXL3 4.05 bpw](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a);
  the paired-bitrate check adds 3.05 bpw.
- **Long prefills:** batch 1, 8-bit cache, FP32 recurrent state, 32,768-token
  prefill chunks, per-layer (n-gram) embeddings streamed from disk, no MTP.
- **API checks:** calibrated head, 8-bit target and draft caches, FP32
  recurrent state, context 32,768 with 98,304 cache tokens, 32,768-token
  prefill chunks, batch 4, MTP with three draft tokens.
- **Paired-bitrate check:** head 1.1.0, context 65,536 with 73,728 cache
  tokens, 8,192-token prefill chunks, batch 4, MTP with three draft tokens.

Run configurations: [integration recipes, samples and counters](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/integration-v1.1.md).

## ExLlamaV3 Coexistence

Generation and decisions share one loaded model in these checks. Recorded idle
counters return to zero.

| Check | Recorded Result |
| --- | --- |
| Overlapping API completions | Four distinct fresh 16,384-token prompts each return HTTP 200 and one output token; API in-flight peak is 4. |
| MTP continuation | A fresh 16,384-token prompt produces 256 tokens, with 190 accepted and 8 rejected draft tokens. |
| Choice API, 8 options | A 2,048-token base yields eight complete, finite calibrated scores. |
| Text and image decisions | Each of 4.05/3.05 bpw completes 24 finite decisions in 12 fresh/retained pairs, with exact paired scores and ranks. |

The MTP continuation stops at its 256-token limit. Each check is one pass.
Cross-bitrate winners agree in 10/12 cases and top-two membership in 9/12.
These checks measure successful requests and retained-prefix consistency.
Decision results were measured on the v1.1 release candidate before the
decision-root scheduling fix; prefill and decode do not use that path.

### Long Prefills

Generation-only ExLlamaV3 probes, with no head loaded, complete exact-length
prefills: a fresh 131,072-token prompt, 512- and 8,192-token appends to 131,072
cached tokens, and an 8,192-token append to 249,856 cached tokens within a
262,144-token context. All are single probes with no measured repeat spread.

## Loaded But Unused Head

ExLlamaV3 loaded-but-unused-head overhead is unmeasured.

## ExLlamaV3 Generation Control

The controls are upstream ExLlamaV3 1.6.2 and master, compared with the FuZev
fork. The fork adds the
[prefill optimizations from exllamav3-fuzev (`exllamav3.optimized`)](https://github.com/PeisongXiao/exllamav3-fuzev/blob/v1.1/doc/fuzev-fork.md#prefill-optimizations);
both sides run generation only. The comparison includes the fork's engine
changes. [Performance](performance.md) reports prefill and
[benchmarks](benchmarks.md#decode) report decode.

## Data

- [Source integration records](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/README.md#v11-integration):
  timing, lifecycle and speculation measurements.
- [Paired-bitrate aggregate](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/v1.1-head/bitrate-api.json):
  text and image decisions on 4.05 and 3.05 bpw.
- [v1.0 integration checks](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/integration-v1.0.md),
  including the vLLM research checks.

## Limitations

- Implementation checks cover the recorded configurations and short workloads.
- ExLlamaV3 measurements use a pre-release fork build. These measurements were
  not repeated on the released build.
- The API and paired-bitrate checks run the application in process through
  ASGI; network serving overhead is unmeasured.
- Image input is experimental. ExLlamaV3 decisions can reuse a retained prefix,
  including its image embeddings, from a generation request that included an image.

See the [API limitations](api.md#limitations) for supported engine settings and
[benchmarks](benchmarks.md) for decision quality.
