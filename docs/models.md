# Models

## Decision Head

One small head handles Choice, yes/no (Noul) and ordered Score. The v1.0 head
has 2,872,401 FP32 parameters in 15 tensors and reads 2,560-wide final hidden
states plus the backbone's label logits. The bundle's `config.json` declares the
geometry, encoding, primitives and maximum option count.

The head is trained on features from
[turboderp/Qwen3.8-Flash-Next-exl3, 4.05 bpw](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a).
The [head card](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev) describes training and quality.
For another backbone, train a matching head with the
[training guide](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/train-a-shared-head.md);
the head's [encoding](#encodings) determines how it is used.

## Encodings

- `fuzev-candidate-branches-v1`: the published head's Qwen chat-format encoding,
  and the only one the v1.0 engine plugins serve. Options use labels A-Z; a head
  with more than 26 options needs a new encoding version.
- `fuzev-hf-text-branches-v1`: the training guide's generic text encoding. Heads
  trained with it load and score through the runtime Python API in v1.0.
- Engine serving of other encodings is planned for v1.1.

## Backbones

The original model is [Qwen3.8-Flash-Next by the Qwen team](https://huggingface.co/Qwen/Qwen3.8-Flash-Next),
under the [Qwen Community License 1.0](https://huggingface.co/Qwen/Qwen3.8-Flash-Next/blob/main/LICENSE).
Measured quantizations (`bpw` = bits per weight):

| Configuration | Checkpoint | Revision |
| --- | --- | --- |
| ExLlamaV3 EXL3 3.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/69e33439ae950f17bcbe95c98f117d80f759ab6d) | `69e33439ae950f17bcbe95c98f117d80f759ab6d` |
| ExLlamaV3 EXL3 4.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a) | `55a732e0c4c3d4614bc42b68493bb930d9b02c0a` |
| ExLlamaV3 EXL3 5.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/7cef615f7fd3681295b68848018876dfabc336c7) | `7cef615f7fd3681295b68848018876dfabc336c7` |
| vLLM NVIDIA NVFP4 | [nvidia/Qwen3.8-Flash-Next-NVFP4](https://huggingface.co/nvidia/Qwen3.8-Flash-Next-NVFP4/tree/fc694b54fb0174e0913e6adf86691ef85a4ead47) | `fc694b54fb0174e0913e6adf86691ef85a4ead47` |
| vLLM RadixArk NVFP4 | [RadixArk/Qwen3.8-Flash-Next-NVFP4](https://huggingface.co/RadixArk/Qwen3.8-Flash-Next-NVFP4/tree/7b719225242aacd3dbd3f9407468c2ee9a9d2594) | `7b719225242aacd3dbd3f9407468c2ee9a9d2594` |
| vLLM EXL3 4.05 bpw | [turboderp/Qwen3.8-Flash-Next-exl3](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a) | `55a732e0c4c3d4614bc42b68493bb930d9b02c0a` |

The [head notices](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev/blob/main/NOTICE)
credit quantizers, backbone terms, training datasets and evaluation sources.

## Use

Install an engine and its decision integration with the
[installation guide](install.md); GB10 installation checks are under
[verified source installations](install.md#verified-source-installations).
The [API reference](api.md) covers SystemOne requests, probabilities,
calibration and retained prefixes.

- Omitted sampling fields use the engine's or model's generation defaults.
- Generation supports [OpenAI-style SSE streaming](api.md#generation-and-discovery)
  with `stream: true`.
- Calibration is a server-startup setting: `raw` by default, or `calibrated` to
  apply the per-type temperatures in the bundle's `calibration.json`.

## Recorded Benchmark Settings

Quality runs use pre-release builds of the archived research evaluation
servers. [Benchmarks](benchmarks.md) and [performance](performance.md) describe
their workloads and measured builds.

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
  [study settings](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/historical-study-settings.json).
  Choose memory and cache settings for your hardware and workload.
- The [integration checks](integration.md) use separately recorded
  configurations and include generation with MTP.

## Licenses

- Apache-2.0: `fuzev`, `fuzev-runtime`, both decision plugins, `vllm-fuzev`
  and the head.
- MIT: `exllamav3-fuzev`.
- AGPL-3.0-only: the optional `vllm-exl3-plugin` quantization integration.
- Backbone and dataset terms are separate ([NOTICE](../NOTICE)).

## Limitations

FuZev v1.0 is an experimental research release.

- The [API limitations](api.md#limitations) cover supported topology, option
  counts and experimental images.
- The published head runs on CPU or CUDA; the serving engines need CUDA.
- The published builds' image path has CPU test coverage only. The image
  results in [benchmarks](benchmarks.md) come from the research evaluation
  servers.
- The three vLLM quality runs use a fork build that lacks two published-build
  changes affecting decision features: CUDA-graph replay of per-layer embeddings
  and KV-cache slot mapping. They are not rerun on the published build.
- The ExLlamaV3 quality runs use a pre-release fork build. Its extension lacks
  the released CUDA-graph kernel-argument fix, and its optimized prefill casts
  gated-delta-network operands to FP16. They are not rerun on the released
  build.
- Research methods are in the
  [archive](https://github.com/PeisongXiao/fuzev-research-archive).
