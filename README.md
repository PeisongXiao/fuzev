# FuZev

**Weights on Hugging Face:** [peisongxiao/Qwen3.8-Flash-Next-FuZev](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev)

FuZev adds Choice, yes/no (Noul) and Score decisions to a vLLM or ExLlamaV3 server
through the Jev SystemOne API. A small shared head reads the model's final hidden
state and label logits alongside normal generation.

## Key Highlights

- **No fine-tuning.** The [backbone](#head-and-backbones) stays frozen. A
  2.9M-parameter head reads its final hidden state and label logits to make
  decisions.
- **Load once, generate and decide.** Text generation and
  [decisions](docs/api.md) share one copy of the weights, one KV cache and one
  scheduler.
- **One head across quantizations and engines.** The same head file runs on
  [vLLM and ExLlamaV3](#use), tested on
  [five quantizations in six engine setups](#six-configurations-5075-requests).
- **Mixture-of-experts backbone.**
  [Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next) activates
  6B of its 125B parameters per token. Kev 1.0's
  [released decision models](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) are all dense.
- **Ahead of Kev-9B with fewer active parameters.** On Kev's held-out
  [`breadth-v1` TEST](#breadth-v1-test), FuZev with Qwen3.8-Flash-Next (6B
  active) scores 46.8 to dense Kev-9B v2's 41.0. Kev-27B and Jev score higher.

> **One machine. One model load. Text generation and decisions.**

**Contents**

- [Use](#use)
  - [vLLM With NVFP4](#vllm-with-nvfp4)
  - [ExLlamaV3 With EXL3](#exllamav3-with-exl3)
  - [Send A Request](#send-a-request)
- [Train Your Own Head](#train-your-own-head)
- [Results](#results)
  - [Six Configurations: 5,075 Requests](#six-configurations-5075-requests)
  - [breadth-v1 TEST](#breadth-v1-test)
- [Components](#components)
- [Head And Backbones](#head-and-backbones)
- [Limitations](#limitations)
- [License](#license)

## Use

Choose vLLM or ExLlamaV3 and run its steps in one shell. Both paths fetch the head from
[peisongxiao/Qwen3.8-Flash-Next-FuZev](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev)
with `hf download`. An existing head directory also works.

### vLLM With NVFP4

Run the [Create An Environment](docs/install.md#create-an-environment),
[vLLM engine](docs/install.md#vllm) and
[vLLM With NVFP4](docs/install.md#vllm-with-nvfp4) installation steps. Then
download the head and start the server. The model downloads on first start.

```bash
hf download peisongxiao/Qwen3.8-Flash-Next-FuZev --local-dir fuzev-head
printf '%s\n' '{"model":"nvidia/Qwen3.8-Flash-Next-NVFP4","async_scheduling":false,"gpu_memory_utilization":0.75,"max_model_len":32768,"max_num_seqs":8,"additional_config":{"ple_storage":"mmap"}}' > engine.json
export VLLM_USE_V2_MODEL_RUNNER=0
fuzev-vllm --engine-config engine.json --model-name my-model --head-directory ./fuzev-head --head-device cpu
```

The JSON holds GB10 example settings. Set `gpu_memory_utilization`,
`max_model_len` and `max_num_seqs` for your GPU.

### ExLlamaV3 With EXL3

Run the [Create An Environment](docs/install.md#create-an-environment),
[ExLlamaV3 engine](docs/install.md#exllamav3) and
[ExLlamaV3 With EXL3](docs/install.md#exllamav3-with-exl3) installation steps.
The build needs the CUDA toolkit (`nvcc`). Then download the head and
turboderp's EXL3 4.05 bpw pack, and start the server.

```bash
hf download peisongxiao/Qwen3.8-Flash-Next-FuZev --local-dir fuzev-head
hf download turboderp/Qwen3.8-Flash-Next-exl3 \
  --revision 55a732e0c4c3d4614bc42b68493bb930d9b02c0a --local-dir qwen-exl3
fuzev-native --model ./qwen-exl3 --head ./fuzev-head --model-name my-model
```

The first request after startup takes a few extra seconds while the prefill
kernels compile.

### Send A Request

Both servers listen on `127.0.0.1:8000`. [Server setup](docs/install.md#start-a-server)
covers listener options; the installation guide also covers
[vLLM with EXL3](docs/install.md#vllm-with-exl3). From another terminal:

```bash
curl --fail-with-body http://127.0.0.1:8000/v1/systemone \
  -H 'Content-Type: application/json' \
  -d '{"state":"Please refund the duplicate charge.","questions":{"refund":{"type":"noul","instructions":"Does the customer request a refund?"}}}'
```

The response holds `model`, the probability `answers.refund.noul` and token `usage`.
The [API reference](docs/api.md) covers the TypeSafe SDK, errors, retained
prefixes and experimental image inputs.

## Train Your Own Head

Train a head for your model of choice with the [training guide](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/train-a-shared-head.md) in
the research archive. The guide extracts features from a frozen Hugging Face
model, trains and calibrates the head, and exports a `fuzev-runtime` bundle.

- Your head loads and scores through the [runtime Python API](https://github.com/PeisongXiao/fuzev-runtime/blob/main/README.md#scoring-from-python).
- In v1.0, the vLLM and ExLlamaV3 servers serve the published head's encoding;
  serving other [encodings](docs/models.md#encodings) is planned for v1.1.

## Results

All runs use NVIDIA GB10, the same 2,872,401-parameter head, raw probabilities
and pre-release builds of the archived research evaluation servers.

- FuZev with Qwen3.8-Flash-Next scores 46.8 [44.2, 50.0] on `breadth-v1`
  TEST: above Kev-0.8B, Kev-4B and Kev-9B v2, below Kev-27B and Jev (unpaired
  intervals). FuZev trains only a head; Kev fine-tunes backbone LoRA adapters or
  the full backbone.
- All six configurations' DEV index intervals overlap.
- Paired against the frozen-LM label control (the backbone's label logits without
  the head), the head gains +4.4 [2.0, 6.4] index points.
- ExLlamaV3 repeats within one server are identical. vLLM repeats shift
  probabilities by up to 0.25, and about 15% of decisions on shared requests differ
  between NVIDIA NVFP4 runs ([repeat probes](docs/benchmarks.md#repeat-probes)).

### Six Configurations: 5,075 Requests

Each run sends 2,000 held-out evaluation-mix requests and 3,075 Kev DEV questions
to the HTTP server ([recorded settings](docs/models.md#recorded-benchmark-settings)).

| Configuration | Checkpoint | Completed Requests | `breadth-v1` DEV Index [95% CI] (3,075 questions) | Evaluation-Mix Selection (2,000) |
| --- | --- | ---: | --- | ---: |
| ExLlamaV3 EXL3 3.05 bpw | [turboderp@69e33439](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/69e33439ae950f17bcbe95c98f117d80f759ab6d) | 5,075/5,075 | 50.5 [48.0, 53.6] | 76.05% |
| ExLlamaV3 EXL3 4.05 bpw | [turboderp@55a732e0](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a) | 5,075/5,075 | 50.0 [47.6, 53.0] | 78.40% |
| ExLlamaV3 EXL3 5.05 bpw | [turboderp@7cef615f](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/7cef615f7fd3681295b68848018876dfabc336c7) | 5,075/5,075 | 50.3 [48.0, 53.4] | 77.25% |
| vLLM NVIDIA NVFP4 | [NVIDIA@fc694b54](https://huggingface.co/nvidia/Qwen3.8-Flash-Next-NVFP4/tree/fc694b54fb0174e0913e6adf86691ef85a4ead47) | 5,075/5,075 | 49.7 [47.2, 52.9] | 77.45% |
| vLLM RadixArk NVFP4 | [RadixArk@7b719225](https://huggingface.co/RadixArk/Qwen3.8-Flash-Next-NVFP4/tree/7b719225242aacd3dbd3f9407468c2ee9a9d2594) | 5,075/5,075 | 52.3 [49.9, 55.4] | 76.45% |
| vLLM EXL3 4.05 bpw | [turboderp@55a732e0](https://huggingface.co/turboderp/Qwen3.8-Flash-Next-exl3/tree/55a732e0c4c3d4614bc42b68493bb930d9b02c0a) | 5,075/5,075 | 50.1 [47.7, 53.3] | 77.80% |

### `breadth-v1` TEST

Kev rows are [Kev 1.0's published results](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models).

| System | Backbone | `breadth-v1` TEST Index | 95% CI |
| --- | --- | ---: | --- |
| [Kev-0.8B](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) | Qwen3.5-0.8B-Base, dense | 23.3 | [21.2, 25.9] |
| [Kev-4B](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) | Qwen3.5-4B-Base, dense | 38.0 | [35.5, 41.3] |
| [Kev-9B v2](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) | Qwen3.5-9B-Base, dense | 41.0 | [38.8, 43.9] |
| **FuZev with Qwen3.8-Flash-Next (ExLlamaV3 + 4.05 bpw)** | Frozen MoE: 125B total, 6B active, plus n-gram embeddings | **46.8** | [44.2, 50.0] |
| [Kev-27B](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) | Qwen3.8-27B, dense | 52.3 | [49.2, 55.4] |
| [Jev](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) | Hosted, undisclosed | 54.0 | [51.2, 57.0] |

Methods, controls, DEV comparisons, lowest results and repeat probes:
[benchmarks](docs/benchmarks.md). Generation with decisions:
[integration checks](docs/integration.md). Timing: [performance](docs/performance.md).

## Components

| Component | Role |
| --- | --- |
| [fuzev-runtime](https://github.com/PeisongXiao/fuzev-runtime) | Head loading, scoring and SystemOne mapping |
| [fuzev-vllm-plugin](https://github.com/PeisongXiao/fuzev-vllm-plugin) | Decision API for a vLLM engine |
| [fuzev-exllamav3-plugin](https://github.com/PeisongXiao/fuzev-exllamav3-plugin) | Decision API for a Generator and the `fuzev-native` launcher |
| [vllm-fuzev](https://github.com/PeisongXiao/vllm-fuzev) | vLLM model and feature-readout hooks |
| [exllamav3-fuzev](https://github.com/PeisongXiao/exllamav3-fuzev) | ExLlamaV3 feature-readout hooks |
| [vllm-exl3-plugin](https://github.com/PeisongXiao/vllm-exl3-plugin) | EXL3 loading in vLLM; AGPL-3.0-only |
| [Research archive](https://github.com/PeisongXiao/fuzev-research-archive) | Training methods, evaluation and reproduction tools |
| [Qwen3.8-Flash-Next-FuZev](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev) | Decision head weights, configuration and calibration (Hugging Face) |

## Head And Backbones

- [Head card](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev): the shared
  2,872,401-parameter head, trained on Qwen3.8-Flash-Next EXL3 4.05 bpw.
- [Models](docs/models.md): backbone checkpoints and tested settings.
- [Releases](docs/releases.md): component versions.

## Limitations

- FuZev v1.0 is an experimental research release.
- **Options:** Choice and Score accept 2 to `max_options` options (16 for the
  published head); Noul has two ([capacity comparison](docs/api.md#capacity-comparison)).
- **Platform:** Tested on NVIDIA GB10 (Python 3.12, PyTorch 2.13.0+cu130). Engines
  need CUDA; the head runs on CPU or CUDA ([verified source installations](docs/install.md#verified-source-installations)).
- **Quality evidence:** Quality runs use pre-release fork builds. The published
  builds have unit, API and installation tests and no end-to-end quality benchmark
  ([benchmark limitations](docs/benchmarks.md#limitations)).
- **Engine settings:** vLLM readout needs the V1 model runner and synchronous
  scheduling, and serves EXL3 checkpoints without MTP. ExLlamaV3 readout runs
  without tensor parallelism ([API limitations](docs/api.md#limitations)).
- **Images:** Experimental and CPU-tested only in the published builds.
  VL-RewardBench preference quality regresses against the backbone control
  ([images](docs/api.md#images)).
- **Repeatability:** vLLM probabilities vary between repeated requests. ExLlamaV3
  probabilities vary slightly across server starts and prefix-cache states
  ([repeat probes](docs/benchmarks.md#repeat-probes)).
- **Measured scope:** Quality is measured on the Results workloads. Quality at the
  server's full context length is unmeasured.

Research methods: [research archive](https://github.com/PeisongXiao/fuzev-research-archive).

## License

Index code and documentation: [Apache-2.0](LICENSE).
Copyright 2026 Peisong Xiao. The head has its own Apache-2.0 license.
[NOTICE](NOTICE) credits backbone and dataset terms.
