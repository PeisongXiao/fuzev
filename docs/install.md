# Install FuZev

Install FuZev from locally downloaded release archives or wheels, then choose
one serving engine. Commands use Bash on Linux.

## Local Files And Requirements

Extract the source archives you need for your engine path into one directory,
using the names below.

| Source Directory | Release | Used By |
| --- | --- | --- |
| `fuzev-runtime` | `1.0.0` | All decision integrations |
| `fuzev-vllm-plugin` | `1.0.0` | vLLM |
| `fuzev-exllamav3-plugin` | `1.0.0` | ExLlamaV3 |
| `vllm-fuzev` | `0.30.0+fuzev.v1.0` | vLLM with NVFP4 or EXL3 |
| `exllamav3-fuzev` | `1.5.2+fuzev.v1.0` | ExLlamaV3 serving and packed EXL3 execution in vLLM |
| `vllm-exl3-plugin` (AGPL-3.0-only) | `0.5.0+fuzev.v1.0` | vLLM with EXL3 |

Requirements:

- Python 3.11 or newer (tested with 3.12) with its venv module.
- A compatible NVIDIA driver and a CUDA-enabled Torch wheel for your platform.
- For ExLlamaV3 builds: Python development headers, a C++ compiler and the CUDA
  toolkit, including `nvcc`.
- Matching CUDA and binary interfaces across toolkit, Torch and engine wheels.

The commands use the tested CUDA 13.0 builds (`cu130` wheel indexes,
`nvidia/cu13` header paths). For other CUDA builds, use the matching
[PyTorch](https://pytorch.org/get-started/locally/) wheel channel and engine
binaries built for that platform.

## Create An Environment

Run this block in a fresh Bash shell, setting the source directory once. Use one
virtual environment per engine installation. For a wheel-only installation, set
`FUZEV_SOURCES` to any local working directory, run the block, then skip to
[Local Release Wheels](#local-release-wheels).

```bash
export FUZEV_SOURCES=/absolute/path/to/extracted-sources
cd "$FUZEV_SOURCES"

python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install "setuptools>=77" wheel packaging ninja

python -m pip install "torch==2.13.0" --index-url https://download.pytorch.org/whl/cu130
export MAX_JOBS=4
```

- The Torch command selects the tested GB10 build.
- `MAX_JOBS` caps parallel compilation for extension builds and first-start
  kernels.
- Keep this environment active, with `MAX_JOBS=4` set, through installation,
  model preparation and server startup.

## Install The Engine

| Serving Path | Engine Installation |
| --- | --- |
| vLLM with NVFP4 | Install the vLLM fork below |
| ExLlamaV3 with EXL3 | Install the ExLlamaV3 fork below |
| vLLM with EXL3 | Install both forks below in the same environment |

Install the downloaded FuZev fork source or its release wheel. The PyPI `vllm`
and `exllamav3` packages are upstream releases and lack the FuZev hooks.

### vLLM

Download the **upstream vLLM 0.30.0 wheel** for your platform, CUDA and Torch
ABI. The fork combines its compiled extensions with the FuZev Python sources.

```bash
python -m pip download --no-deps --only-binary=:all: \
  --dest "$FUZEV_SOURCES/wheels" "vllm==0.30.0"
export FUZEV_VLLM_BINARY_WHEEL="$(python -c \
  'import os; from pathlib import Path; print(next((Path(os.environ["FUZEV_SOURCES"]) / "wheels").glob("vllm-0.30.0-*.whl")).resolve())')"
VLLM_USE_PRECOMPILED=1 \
VLLM_PRECOMPILED_WHEEL_LOCATION="$FUZEV_VLLM_BINARY_WHEEL" \
python -m pip install "$FUZEV_SOURCES/vllm-fuzev"
python -m pip install \
  "flashinfer-jit-cache==$(python -c 'from importlib.metadata import version; print(version("flashinfer-python"))')" \
  --index-url https://flashinfer.ai/whl/cu130
```

- Always set `VLLM_PRECOMPILED_WHEEL_LOCATION` in this build mode. It selects
  the matching release binaries.
- The FlashInfer cache holds prebuilt kernels matching the installed
  `flashinfer-python`. Install it from the `cuXXX` index matching your Torch
  build before starting an engine.

### ExLlamaV3

Build `exllamav3_ext` against the active environment's Torch. For the tested
CUDA 13 build, first add pip's NVIDIA header directory:

```bash
export FUZEV_CUDA_INCLUDE="$(python -c 'import sysconfig; print(sysconfig.get_paths()["purelib"])')/nvidia/cu13/include"
unset CPATH C_INCLUDE_PATH CPLUS_INCLUDE_PATH
export CPLUS_INCLUDE_PATH="$FUZEV_CUDA_INCLUDE"
export LIBRARY_PATH="$(python -c 'import sysconfig; print(sysconfig.get_paths()["purelib"])')/nvidia/cu13/lib"
```

- The toolkit's headers take precedence over these supplemental wheel headers.
- For a system-only toolkit, clear these inherited include variables and use
  the toolkit's normal header and library search paths.
- To pick the build architecture, set `TORCH_CUDA_ARCH_LIST` to your GPU's
  compute capability (`12.1` on the tested GB10). Otherwise Torch can detect
  the locally visible GPU.

```bash
python -m pip install --no-build-isolation "$FUZEV_SOURCES/exllamav3-fuzev"
python -m pip install \
  "flashinfer-jit-cache==$(python -c 'from importlib.metadata import version; print(version("flashinfer-python"))')" \
  --index-url https://flashinfer.ai/whl/cu130
```

Build isolation is off for this extension build so it uses the installed CUDA
Torch and build tools. Check the build:

```bash
python -c "import torch, exllamav3_ext, importlib.metadata as m; print(m.version('exllamav3'))"
```

Expected output: `1.5.2+fuzev.v1.0`.

## Install The Decision Integration

After installing the engine, run the command for your serving path.

### vLLM With NVFP4

```bash
python -m pip install \
  "$FUZEV_SOURCES/fuzev-runtime" \
  "$FUZEV_SOURCES/fuzev-vllm-plugin"
```

### ExLlamaV3 With EXL3

```bash
python -m pip install \
  "$FUZEV_SOURCES/fuzev-runtime" \
  "$FUZEV_SOURCES/fuzev-exllamav3-plugin"
```

### vLLM With EXL3

With both engine forks installed, install the runtime, decision integration
and quantization plugin together:

```bash
python -m pip install \
  "$FUZEV_SOURCES/fuzev-runtime" \
  "$FUZEV_SOURCES/fuzev-vllm-plugin" \
  "$FUZEV_SOURCES/vllm-exl3-plugin"

export VLLM_PLUGINS=vllm_exl3
```

- `VLLM_PLUGINS=vllm_exl3` selects this quantization plugin so no other
  installed plugin takes over EXL3 registration. Keep it set in the shell that
  starts vLLM.
- The default Python-only build runs packed execution on `exllamav3_ext`. For
  `vllm_exl3_c`, follow the [optional native build](https://github.com/PeisongXiao/vllm-exl3-plugin/blob/main/docs/fuzev-serving.md#optional-native-build)
  in the AGPL-3.0-only quantization plugin's guide.
- Packed tables use resident storage by default. The [disk-backed table example](https://github.com/PeisongXiao/vllm-exl3-plugin/blob/main/docs/fuzev-serving.md#disk-backed-packed-tables)
  includes the required lazy loading and CUDA graph settings.

For a Qwen EXL3 pack, [download the model](#head-and-model-files) to
`$FUZEV_MODEL` first. Then create a separate serving view in a new directory
outside the model directory:

```bash
export FUZEV_EXL3_VIEW=/absolute/path/to/new-serving-view

python "$FUZEV_SOURCES/vllm-exl3-plugin/tools/exl3_pack_tools/qwen_pack_view.py" \
  "$FUZEV_MODEL" "$FUZEV_EXL3_VIEW"
```

The tool writes a complete regular index, including vision and n-gram embedding
files, and preserves the original model's index.

Configure vLLM with the serving-view directory, `quantization="exl3"` and
`engram_config: {"cpu_offload": false}`. The FuZev vLLM fork includes the Qwen
EXL3 model changes. The quantization plugin's
[serving guide](https://github.com/PeisongXiao/vllm-exl3-plugin/blob/main/docs/fuzev-serving.md#configure-vllm)
gives complete commands.

### Local Release Wheels

Local FuZev release wheels can replace the source builds above. Put only the
wheels for one engine stack, matching your platform and Torch/CUDA build, in a
separate directory:

- The FuZev engine fork wheel, the runtime and the decision integration.
- For vLLM with EXL3, both engine forks and the quantization plugin.

Keep the upstream vLLM wheel from the source build outside this directory.

```bash
export FUZEV_WHEELS=/absolute/path/to/selected-release-wheels
python -m pip install "$FUZEV_WHEELS"/*.whl
python -m pip install \
  "flashinfer-jit-cache==$(python -c 'from importlib.metadata import version; print(version("flashinfer-python"))')" \
  --index-url https://flashinfer.ai/whl/cu130
```

Pip resolves remaining dependencies from its configured indexes.

## Head And Model Files

Serving decisions requires a compatible model and a local head bundle with
`config.json`, `head.safetensors` and `calibration.json`.
[Model information](models.md) lists the supported backbone and separate model
licenses.

Download and verify the [published head](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev).
For a head already on disk, set `FUZEV_HEAD` to it and run only the check:

```bash
export FUZEV_HEAD="$PWD/fuzev-head"
hf download peisongxiao/Qwen3.8-Flash-Next-FuZev --local-dir "$FUZEV_HEAD"
(cd "$FUZEV_HEAD" && sha256sum -c SHA256SUMS)
```

vLLM with NVFP4 downloads its model on first start. For ExLlamaV3 and vLLM
with EXL3, download turboderp's EXL3 4.05 bpw pack at the head's training
revision:

```bash
export FUZEV_MODEL="$PWD/qwen3.8-flash-next-exl3-4.05bpw"
hf download turboderp/Qwen3.8-Flash-Next-exl3 \
  --revision 55a732e0c4c3d4614bc42b68493bb930d9b02c0a --local-dir "$FUZEV_MODEL"
```

## Start A Server

In the active serving environment, save this vLLM NVFP4 example as `engine.json`:

```json
{
  "model": "nvidia/Qwen3.8-Flash-Next-NVFP4",
  "async_scheduling": false,
  "gpu_memory_utilization": 0.75,
  "max_model_len": 32768,
  "max_num_seqs": 8,
  "additional_config": {"ple_storage": "mmap"}
}
```

- Memory fraction (0.75), context window and sequence count are example
  operator settings for GB10.
- On unified-memory GPUs such as GB10, `gpu_memory_utilization` reserves a share
  of system RAM. Leave room for the OS, the head and kernel compilation.
- `ple_storage: mmap` reads Qwen3.8's per-layer n-gram embedding tables from the
  memory-mapped checkpoint and selects PIECEWISE CUDA graphs.
- Decision readout uses synchronous scheduling (`async_scheduling: false`) and
  the V1 model runner.

Start vLLM:

```bash
export VLLM_USE_V2_MODEL_RUNNER=0
fuzev-vllm --engine-config engine.json --model-name my-model \
  --head-directory "$FUZEV_HEAD" --head-device cpu
```

For ExLlamaV3, start the server with the downloaded model:

```bash
fuzev-native --model "$FUZEV_MODEL" --head "$FUZEV_HEAD" --model-name my-model
```

Both launchers accept `--host`, `--port` and `--api-key`, and default to
`127.0.0.1:8000` with no key. Send a request from another shell:

```bash
curl --fail-with-body http://127.0.0.1:8000/v1/systemone \
  -H 'Content-Type: application/json' \
  -d '{"state":"Please refund the duplicate charge.","questions":{"refund":{"type":"noul","instructions":"Does the customer request a refund?"}}}'
```

HTTP 200 returns the configured `model`, `answers.refund.noul` (the Yes
probability), `usage` and `latency_ms`. Probabilities depend on the model and
head; the [API guide](api.md#usage) shows an illustrative response. On
ExLlamaV3, the first request after startup takes a few seconds longer while
prefill kernels compile.

## Verified Source Installations

Each source-ZIP configuration below returns HTTP 200 for one combined Noul,
Choice and Score request on GB10, with Python 3.12.3, PyTorch 2.13.0+cu130
(CUDA 13.0), `MAX_JOBS=4` and the head on CPU.

| Package | ExLlamaV3 EXL3 | vLLM NVFP4 | vLLM EXL3 |
| --- | --- | --- | --- |
| Engine | `exllamav3 1.5.2+fuzev.v1.0` | `vllm 0.30.0+fuzev.v1.0.precompiled` | `vllm 0.30.0+fuzev.v1.0.precompiled` with `exllamav3 1.5.2+fuzev.v1.0` |
| Runtime | `fuzev-runtime 1.0.0` | `fuzev-runtime 1.0.0` | `fuzev-runtime 1.0.0` |
| Decision integration | `fuzev-exllamav3-plugin 1.0.0` | `fuzev-vllm-plugin 1.0.0` | `fuzev-vllm-plugin 1.0.0` |
| Quantization integration | Engine EXL3 kernels | Engine NVFP4 kernels | `vllm-exl3 0.5.0+fuzev.v1.0`, Python plugin |
| FlashInfer | `0.7.1` | `0.6.18.post1` | `0.6.18.post1` |
| Matching JIT cache | `0.7.1+cu130` | `0.6.18.post1+cu130` | `0.6.18.post1+cu130` |
| Peak memory | 77.20 GiB | 98.72 GiB | 97.03 GiB |

Use the environment, engine and integration blocks above. Peak memory is
the highest sampled host-wide memory use. The
[installation record](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/component-install-smoke.md#source-zip-serving-checks)
holds each path's executed build/install commands, source versions, complete
settings, request and response.

### ExLlamaV3 EXL3

The recorded launcher serves turboderp's Qwen3.8-Flash-Next EXL3 4.05 bpw pack:

- 32,768-token context, 65,536-token 8-bit KV cache, batch size 8.
- Default prefill optimizations and default 8,192-token prefill chunk.
- MTP and vision off.

```bash
fuzev-native --model "$FUZEV_MODEL" --model-name qwen-flash-next-exl3 \
  --head "$FUZEV_HEAD" --head-device cpu \
  --context-length 32768 --cache-tokens 65536 --cache-bits 8 8 \
  --max-batch-size 8
```

The [ExLlamaV3 record](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/configs/smoke-native-exl3.json)
contains the exact request and response.

### vLLM NVFP4

The recorded configuration is the `engine.json` example above, with default
BF16 KV, PIECEWISE CUDA graphs and MTP off. It sets no batch-token budget; vLLM
selects 2,048 batched tokens on GB10.

```bash
export VLLM_USE_V2_MODEL_RUNNER=0
fuzev-vllm --engine-config engine.json \
  --model-name nvidia/Qwen3.8-Flash-Next-NVFP4 \
  --head-directory "$FUZEV_HEAD" --head-device cpu
```

The [NVFP4 record](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/configs/smoke-vllm-nvfp4.json)
contains the exact request and response.

### vLLM EXL3

The recorded configuration uses the complete
[disk-backed packed-table JSON](https://github.com/PeisongXiao/vllm-exl3-plugin/blob/main/docs/fuzev-serving.md#disk-backed-packed-tables)
as `exl3-engine.json`, with the serving view of turboderp's 4.05 bpw pack in
`model`:

- Memory fraction 0.75, 8,192-token context, 4 sequences (`max_num_seqs`).
- Synchronous scheduling, lazy safetensors loading, default BF16 KV, MTP off.
- Top-level `engram_config.cpu_offload: false` lets the quantization plugin
  manage n-gram embedding storage.
- Disk-backed tables with PIECEWISE CUDA graphs.
- Default Python-only quantization plugin build.
- ExLlamaV3 fork installed from a wheel built in a separate
  [ExLlamaV3](#exllamav3) environment.

```bash
export VLLM_USE_V2_MODEL_RUNNER=0
export VLLM_PLUGINS=vllm_exl3
export VLLM_EXL3_NGRAM_TABLE=disk
fuzev-vllm --engine-config exl3-engine.json --model-name qwen-flash-next-exl3 \
  --head-directory "$FUZEV_HEAD" --head-device cpu
```

The [vLLM EXL3 record](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/configs/smoke-vllm-exl3-4.05.json)
contains the exact request and response.

## Limitations

- Tested on NVIDIA GB10 (Python 3.12, PyTorch 2.13.0+cu130).
- vLLM with EXL3 and MTP is unsupported.
- Hardware capacity, concurrency and memory allocation follow the chosen
  engine's settings.
- GPU image serving is experimental and has CPU-only request coverage.
