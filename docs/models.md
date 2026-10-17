# Models

## Decision Head

One small head handles Choice, yes/no (Noul) and ordered Score. Head 1.1.0
has 2,872,401 FP32 parameters in 15 tensors and reads 2,560-wide final hidden
states plus the backbone's label logits. The bundle's `config.json` declares the
geometry, encoding and maximum option count for each primitive and modality.
Head 1.1.0 declares `implemented_max`, the most options a request may have, and
`validated_max`, the most options with held-out quality results (null when
unvalidated):

| Primitive | Modality | Implemented Max | Validated Max |
| --- | --- | ---: | ---: |
| Choice | text | 255 | 255 |
| Noul | text | 2 | 2 |
| Ordinal (Score) | text | 255 | unvalidated |
| Choice | image_text | 26 | 2 |
| Noul | image_text | 2 | unvalidated |
| Ordinal (Score) | image_text | 26 | unvalidated |

The head is trained on features from
[turboderp/Qwen3.8-Flash-Next-exl3, 4.05 bpw](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a).
The [head card](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev/tree/v1.1) describes training and quality.
For another backbone, train a matching head with the
[training guide](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/train-a-shared-head.md);
the head's [encoding](#encodings) determines how it is used.

## Encodings

- `fuzev-candidate-branches-v1`: head 1.0.0's Qwen chat-format encoding,
  served by the ExLlamaV3 plugin. Options use labels A-Z; a head
  with more than 26 options needs a new encoding version.
- `fuzev-list-aware-v2`: head 1.1.0's encoding, served by the ExLlamaV3 plugin.
  Lists of up to 26 options use the same A-Z labels and label logits; longer
  lists are numbered and use no label logits.
- `fuzev-hf-text-branches-v1`: the training guide's generic text encoding. Heads
  trained with it load and score through the runtime Python API in v1.1.

## Backbones

The original model is [Qwen3.8-Flash-Next by the Qwen team](https://huggingface.co/Qwen/Qwen3.8-Flash-Next),
under the [Qwen Community License 1.0](https://huggingface.co/Qwen/Qwen3.8-Flash-Next/blob/main/LICENSE).
FuZev v1.1 serves the EXL3 checkpoints with ExLlamaV3; the vLLM rows come from
head 1.0.0's [cross-engine research study](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/benchmarks-v1.0.md).
Measured quantizations (`bpw` = bits per weight):

| Configuration | Checkpoint | Revision |
| --- | --- | --- |
| ExLlamaV3 EXL3 3.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/69e33439ae950f17bcbe95c98f117d80f759ab6d) | `69e33439ae950f17bcbe95c98f117d80f759ab6d` |
| ExLlamaV3 EXL3 4.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a) | `55a732e0c4c3d4614bc42b68493bb930d9b02c0a` |
| ExLlamaV3 EXL3 5.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/7cef615f7fd3681295b68848018876dfabc336c7) | `7cef615f7fd3681295b68848018876dfabc336c7` |
| vLLM NVIDIA NVFP4 | [nvidia/Qwen3.8-Flash-Next-NVFP4](https://huggingface.co/nvidia/Qwen3.8-Flash-Next-NVFP4/tree/fc694b54fb0174e0913e6adf86691ef85a4ead47) | `fc694b54fb0174e0913e6adf86691ef85a4ead47` |
| vLLM RadixArk NVFP4 | [RadixArk/Qwen3.8-Flash-Next-NVFP4](https://huggingface.co/RadixArk/Qwen3.8-Flash-Next-NVFP4/tree/7b719225242aacd3dbd3f9407468c2ee9a9d2594) | `7b719225242aacd3dbd3f9407468c2ee9a9d2594` |
| vLLM EXL3 4.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a) | `55a732e0c4c3d4614bc42b68493bb930d9b02c0a` |

The [head notices](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev/blob/v1.1/NOTICE)
credit quantizers, backbone terms, training datasets and evaluation sources.

## Use

Install ExLlamaV3 and its decision integration with the
[installation guide](install.md); GB10 installation checks are under
[verified source installations](install.md#verified-source-installations).
The [API reference](api.md) covers SystemOne requests, probabilities,
calibration and retained prefixes.

- Omitted sampling fields use the engine's or model's generation defaults.
- Generation supports [OpenAI-style SSE streaming](api.md#generation-and-discovery)
  with `stream: true`.
- Calibration is a server-startup setting: `raw` by default, or `calibrated` to
  apply the temperatures in the bundle's `calibration.json`.

## Recorded Benchmark Settings

Head 1.0.0's cross-engine research study uses pre-release builds of the
archived research evaluation servers. [Benchmarks](benchmarks.md) describes its
workloads and measured builds.

| Engine / Format | Context Tokens | KV Cache | Embedding Storage | MTP |
| --- | ---: | --- | --- | --- |
| ExLlamaV3 EXL3 3.05 / 4.05 / 5.05 bpw | 262,144 | FP16, 294,912-token pool | NVMe streaming | Off |
| vLLM NVIDIA / RadixArk NVFP4 | 262,144 | `auto` | Memory-mapped | Off |
| vLLM EXL3 4.05 bpw | 262,144 | `auto` | Resident | Off |

- ExLlamaV3 quality runs score up to eight answer options per forward batch.
  The published ExLlamaV3 plugin submits each option as its own Generator job.
- vLLM batches readout under its scheduler settings. Both vLLM rows use eager
  mode, 4 sequences (`max_num_seqs`), 16,384 batched tokens and prefix caching.
- vLLM EXL3 uses a repacked serving view of turboderp revision `55a732e0` and
  runs packed operations on ExLlamaV3's kernels, without the optional
  `vllm_exl3_c` extension.
- Recorded memory fractions are in the
  [study settings](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/historical-study-settings.json).
  Choose memory and cache settings for your hardware and workload.
- The [integration checks](integration.md) use separately recorded
  configurations and include generation with MTP.

## Licenses

- Apache-2.0: `fuzev`, `fuzev-runtime`, `fuzev-exllamav3-plugin` and the head.
- MIT: `exllamav3-fuzev` (ExLlamaV3 by turboderp, FuZev changes by Peisong Xiao).
- Backbone and dataset terms are separate ([NOTICE](../NOTICE)).

## Limitations

FuZev v1.1 is an experimental research release.

- The [API limitations](api.md#limitations) cover supported topology, option
  counts and experimental images.
- The published head runs on CPU or CUDA; ExLlamaV3 needs CUDA.
- Image decisions are experimental. The image results in
  [benchmarks](benchmarks.md) include tracks where head 1.1.0 scores below the
  frozen-LM label control.
- The cross-engine study's three vLLM quality runs use a fork build that lacks
  two published-build changes affecting decision features: CUDA-graph replay of
  per-layer embeddings and KV-cache slot mapping. They are not rerun on the
  published build.
- The cross-engine study's ExLlamaV3 quality runs use a pre-release fork build.
  Its extension lacks the released CUDA-graph kernel-argument fix, and its
  optimized prefill casts gated-delta-network operands to FP16. They are not
  rerun on the released build.
- Research methods are in the
  [archive](https://github.com/PeisongXiao/fuzev-research-archive/tree/v1.1).
