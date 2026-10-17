# Releases

FuZev components use version `1.1.0`. The engine fork appends `+fuzev.v1.1`
to its upstream version.

| Component | Version |
| --- | --- |
| `fuzev` | `1.1.0` |
| `fuzev-runtime` | `1.1.0` |
| `fuzev-exllamav3-plugin` | `1.1.0` |
| `fuzev-research-archive` | `1.1.0` |
| `exllamav3-fuzev` | `1.6.1+fuzev.v1.1` |
| Decision head (`peisongxiao/Qwen3.8-Flash-Next-FuZev`) | `1.1.0` (Hugging Face tag `v1.1`) |

## Use

- Each component ships a `VERSION` file and package metadata.
- The [installation guide](install.md) covers ExLlamaV3. Its canonical
  [source-installation checks](install.md#verified-source-installations) link
  the engine configuration and HTTP results.
- Results and workloads: [benchmarks](benchmarks.md),
  [integration checks](integration.md) and [performance](performance.md).

## What's New In v1.1

[Upgrading From 1.0](#upgrading-from-10) covers compatibility, changed
interfaces and defaults.

- ExLlamaV3 is the only serving engine. fuzev-vllm-plugin, vllm-fuzev and
  vllm-exl3-plugin stay at 1.0
  ([1.0 installation](https://github.com/PeisongXiao/fuzev/blob/v1.0/docs/install.md)).
- exllamav3-fuzev is based on ExLlamaV3 1.6.1 (1.5.2 in FuZev 1.0)
  ([change map](https://github.com/PeisongXiao/exllamav3-fuzev/blob/v1.1/doc/fuzev-fork.md#change-map)).
- The installation guide downloads the `v1.1` source archives; FuZev
  publishes no release wheels
  ([local files](install.md#local-files-and-requirements)).
- Head 1.1.0 (`--revision v1.1`) accepts 2 to 255 text options and 2 to 26
  image options for Choice and Score (16 in head 1.0.0)
  ([capacity comparison](api.md#capacity-comparison)).
- Head 1.1.0 uses the `fuzev-list-aware-v2` encoding, which numbers lists of
  more than 26 options. The ExLlamaV3 plugin serves it and
  `fuzev-candidate-branches-v1` ([encodings](models.md#encodings)).
- A decision reads its context and question once; each option then adds only
  its own tokens, up to `--max-batch-size` options at a time
  ([decision cost](api.md#limitations)).
- Calibrated mode uses a temperature per question type, modality and
  option-count range. Requests outside the bundle's ranges use raw
  probabilities and report `calibration_id: "raw"`
  ([calibration](api.md#answers)).
- `fuzev-native --prefill-profile` sets prefill options from a JSON object or
  file before the model loads ([settings](performance.md#settings)).
- The v1.0
  [quality study](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/benchmarks-v1.0.md),
  [integration checks](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/integration-v1.0.md)
  and [short-workload timings](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/performance-v1.0.md)
  are in the research archive.

Measured results:

- `breadth-v1` TEST scores come from served runs through an HTTP server
  ([setup](benchmarks.md#setup)), where FuZev v1.0 scores 45.57 [42.95,
  48.67]. FuZev 1.0's published 46.8 [44.2, 50.0] is an offline score of the
  1.0 head
  ([v1.0 quality study](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/benchmarks-v1.0.md#test-controls-and-datasets)).
- Text decision quality is unchanged from FuZev v1.0 overall within the
  paired 95% CI (v1.1 minus v1.0: DEV -2.84 to +2.89, TEST -3.35 to +2.51
  index points); CLINC150 drops 8.67 points on both splits
  ([paired differences](benchmarks.md#paired-differences)).
- The v1.1 decision benchmarks run with `--calibration calibrated`; the
  server default is `raw` ([setup](benchmarks.md#setup)).
- On held-out WANDS, head 1.1.0 ranks an acceptable product first in 24 of 35
  requests at 255 options (uniform control 2/35)
  ([held-out WANDS](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev/blob/v1.1/README.md#held-out-wands)).
- Experimental image decisions with head 1.1.0 score below FuZev v1.0 on
  MM-RLHF: 56.67% against 73.33% on best response and 80.80% against 88.80%
  on pairwise preference
  ([image results](benchmarks.md#experimental-image-decisions)).
- On one NVIDIA GB10 (128 GB unified memory), FuZev v1.1 prefill throughput
  ranges from 4.7% below FuZev v1.0 at 32,768 prompt tokens to 9.2% above it
  at 2,048 tokens ([measurements](performance.md#measurements)). Settings:
  batch 1, Q8 KV cache, 32,768-token chunks, FP32 recurrent state, MTP off;
  Python 3.12.3, PyTorch 2.13.0+cu130, Triton 3.7.1, FlashInfer 0.6.18.post1,
  exllamav3-fuzev 1.6.1+fuzev.v1.1 and 1.5.2+fuzev.v1.0.

Decision results were measured on the v1.1 release candidate before the
decision-root scheduling fix; prefill and decode do not use that path.

## Upgrading From 1.0

- Install fuzev-runtime 1.1.0, fuzev-exllamav3-plugin 1.1.0 and
  exllamav3-fuzev 1.6.1+fuzev.v1.1 together in a fresh environment
  ([installation](install.md)).
- fuzev-exllamav3-plugin 1.0 and fuzev-vllm-plugin 1.0 work with
  fuzev-runtime 1.0 only ([1.0 installation](https://github.com/PeisongXiao/fuzev/blob/v1.0/docs/install.md)).
- `exllamav3.optimized` holds `settings.py` and `profile.json`; the fork has
  no `exllamav3.optimized.install`, `.hooks` or `.vendor` modules.
- Optimized prefill is on by default. `enable_prefill_optimizations(options)`
  updates options before model construction.
  `EXLLAMAV3_NO_PREFILL_OPTIMIZATIONS=1` before `Config.from_directory`, or
  `config.infer_params.prefill_optimizations = False` before
  `Model.from_config`, selects plain ExLlamaV3 prefill.
- Defaults: prefill chunk 32,768 tokens (8,192 in 1.0), capped at the cache
  tokens (`--max-chunk-size` overrides), `moe_batch_cap` 4 (8 in 1.0), FP32
  recurrent state.
- `usage.input_tokens` counts the encoded request once, including shared
  SystemOne state once across questions. `/v1/decisions` reports each decision's
  length in `execution.input_tokens` and its logical branch work in
  `execution.submitted_tokens`.
- Head 1.1.0 downloads with `--revision v1.1`. Head 1.0.0 is at
  `--revision v1.0`, loads in runtime 1.1 and admits up to 16 options.
  Runtime 1.0 loads head 1.0.0 only.

## Limitations

FuZev v1.1 is an experimental research release. The
[API limitations](api.md#limitations) cover option counts, engine scope and
images. Research methods are in the
[archive](https://github.com/PeisongXiao/fuzev-research-archive/tree/v1.1).
