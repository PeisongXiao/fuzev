# Install FuZev

Install FuZev with ExLlamaV3 from the release source archives. Commands use
Bash on Linux.

## Local Files And Requirements

Download and extract the source archives into one directory, using the names
below. Run this and the remaining installation blocks in one Bash shell:

```bash
export FUZEV_SOURCES="$PWD/fuzev-sources"
mkdir -p "$FUZEV_SOURCES"
for repository in fuzev-runtime fuzev-exllamav3-plugin exllamav3-fuzev; do
  curl --fail --location \
    "https://github.com/PeisongXiao/$repository/archive/refs/tags/v1.1.tar.gz" \
    --output "$FUZEV_SOURCES/$repository.tar.gz"
  mkdir -p "$FUZEV_SOURCES/$repository"
  tar -xzf "$FUZEV_SOURCES/$repository.tar.gz" \
    --strip-components=1 --directory "$FUZEV_SOURCES/$repository"
done
```

| Source Directory | Release | Used By |
| --- | --- | --- |
| `fuzev-runtime` | `1.1.0` | All decision integrations |
| `fuzev-exllamav3-plugin` | `1.1.0` | ExLlamaV3 |
| `exllamav3-fuzev` | `1.6.1+fuzev.v1.1` | ExLlamaV3 serving |

Requirements:

- Python 3.11 or newer (tested with 3.12) with its venv module.
- A compatible NVIDIA driver and a CUDA-enabled Torch wheel for your platform.
- NVIDIA driver 550.54.14 or newer on Linux (551.61 or newer on Windows) for
  CUDA graph replay; older drivers run without CUDA graphs.
- For ExLlamaV3 builds: Python development headers, a C++ compiler and the CUDA
  toolkit 12.4 or newer, including `nvcc`.
- Matching CUDA and binary interfaces across toolkit, Torch and the engine build.

The commands use the tested CUDA 13.0 builds (`cu130` wheel indexes,
`nvidia/cu13` header paths). For other CUDA builds, use the matching
[PyTorch](https://pytorch.org/get-started/locally/) wheel channel and engine
binaries built for that platform.

## Create An Environment

Use your CUDA toolkit's `bin` directory and one virtual environment per engine
installation.

```bash
cd "$FUZEV_SOURCES"

export PATH=/usr/local/cuda-13.0/bin:$PATH
nvcc --version
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install "setuptools>=77" wheel packaging ninja

python -m pip install "torch==2.13.0" --index-url https://download.pytorch.org/whl/cu130
export MAX_JOBS=4
```

- The Torch command selects the tested GB10 build.
- `MAX_JOBS` caps parallel compilation for extension builds and first-start
  kernels. 4 is the tested GB10 value; lower it if compilation runs out of
  memory and raise it on hosts with more memory.
- Keep this environment active, with `MAX_JOBS` set, through installation,
  model preparation and server startup.

## Install The Engine

Install the downloaded FuZev fork source. The PyPI `exllamav3` package is an
upstream release and lacks the FuZev hooks.

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

The FlashInfer cache holds prebuilt kernels matching the installed
`flashinfer-python`. Install it from the `cuXXX` index matching your Torch
build before starting an engine.

Build isolation is off for this extension build so it uses the installed CUDA
Torch and build tools. Check the build:

```bash
python -c "import torch, exllamav3_ext, importlib.metadata as m; print(m.version('exllamav3'))"
```

Expected output: `1.6.1+fuzev.v1.1`.

## Install The Decision Integration

After installing the engine, run the command for your serving path.

### ExLlamaV3 With EXL3

```bash
python -m pip install \
  "$FUZEV_SOURCES/fuzev-runtime" \
  "$FUZEV_SOURCES/fuzev-exllamav3-plugin"
```

## Head And Model Files

Serving decisions requires a compatible model and a local head bundle with
`config.json`, `head.safetensors` and `calibration.json`.
[Model information](models.md) lists the supported backbone and separate model
licenses.

Download and verify the [published head](https://huggingface.co/peisongxiao/Qwen3.8-Flash-Next-FuZev/tree/v1.1).
For a head already on disk, set `FUZEV_HEAD` to it and run only the check:

```bash
export FUZEV_HEAD="$PWD/fuzev-head"
hf download peisongxiao/Qwen3.8-Flash-Next-FuZev --revision v1.1 --local-dir "$FUZEV_HEAD"
(cd "$FUZEV_HEAD" && sha256sum -c SHA256SUMS)
```

Download turboderp's EXL3 4.05 bpw pack at the head's training revision:

```bash
export FUZEV_MODEL="$PWD/qwen3.8-flash-next-exl3-4.05bpw"
hf download turboderp/Qwen3.8-Flash-Next-exl3 \
  --revision 55a732e0c4c3d4614bc42b68493bb930d9b02c0a --local-dir "$FUZEV_MODEL"
```

## Start A Server

In the active serving environment, start the server with the downloaded model:

```bash
fuzev-native --model "$FUZEV_MODEL" --head "$FUZEV_HEAD" --model-name my-model \
  --calibration calibrated
```

Calibrated mode applies a matching temperature for the question type, modality
and option count; requests without one use raw probabilities. The default is raw.

`--prefill-profile` takes a JSON object, or the path of a JSON file, with prefill
options and applies them before the model loads; the options and their
GB10-tuned defaults are listed under [performance settings](performance.md#settings).
`--max-chunk-size` overrides the profile's `prefill_chunk`.

The launcher accepts `--host`, `--port` and `--api-key`, and defaults to
`127.0.0.1:8000` with no key. Send a request from another shell:

```bash
curl --fail-with-body http://127.0.0.1:8000/v1/systemone \
  -H 'Content-Type: application/json' \
  -d '{"state":"Please refund the duplicate charge.","questions":{"refund":{"type":"noul","instructions":"Does the customer request a refund?"}}}'
```

HTTP 200 returns the configured `model`, `answers.refund.noul` (the Yes
probability), `usage` and `latency_ms`. Probabilities depend on the model and
head; the [API guide](api.md#usage) shows an illustrative response. On
ExLlamaV3, the first request after startup is slower while prefill kernels
compile.

## Verified Source Installations

The source-archive configuration below, recorded on the v1.1 release
candidate, returns HTTP 200 for one combined Noul, Choice and Score request on
GB10, with Python 3.12.3 and PyTorch 2.13.0+cu130 (CUDA 13.0).

| Package | ExLlamaV3 EXL3 |
| --- | --- |
| Engine | `exllamav3 1.6.1+fuzev.v1.1` |
| Runtime | `fuzev-runtime 1.1.0` |
| Decision integration | `fuzev-exllamav3-plugin 1.1.0` |
| Quantization integration | Engine EXL3 kernels |
| FlashInfer | `0.7.2` |
| Matching JIT cache | `0.7.2+cu130` |

Use the environment, engine and integration blocks above. The
[installation record](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/component-install-smoke.md#v11-source-installation)
holds the executed build/install commands, source versions, settings and check
results.

### ExLlamaV3 EXL3

The recorded launcher serves turboderp's Qwen3.8-Flash-Next EXL3 4.05 bpw pack:

- Launcher defaults: 262,143-token context, FP16 KV cache, batch size 16.
- Default prefill optimizations and default 32,768-token prefill chunk.
- Calibrated probabilities.
- MTP and vision off.

```bash
fuzev-native --model "$FUZEV_MODEL" --head "$FUZEV_HEAD" --model-name my-model \
  --calibration calibrated
```

The [ExLlamaV3 record](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/integration/v1.1/source-installation.json)
lists each check and its HTTP status. Choice requests with 64 and 255 options
return HTTP 200; their selected options differ from the check's synthetic
expected answers, and the record keeps these failures.

## Limitations

- Tested on NVIDIA GB10 (Python 3.12, PyTorch 2.13.0+cu130).
- Hardware capacity, concurrency and memory allocation follow ExLlamaV3's
  settings.
- GPU image serving is experimental. Measured image results, including tracks
  where the head scores below the frozen-LM label control, are in
  [benchmarks](benchmarks.md#experimental-image-decisions); image Choice is
  validated at 2 options.
