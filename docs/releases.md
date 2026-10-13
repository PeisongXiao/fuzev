# Releases

FuZev components use version `1.0.0`. Engine and quantization forks append
`+fuzev.v1.0` to their upstream version.

| Component | Version |
| --- | --- |
| `fuzev` | `1.0.0` |
| `fuzev-runtime` | `1.0.0` |
| `fuzev-vllm-plugin` | `1.0.0` |
| `fuzev-exllamav3-plugin` | `1.0.0` |
| `fuzev-research-archive` | `1.0.0` |
| `vllm-fuzev` | `0.30.0+fuzev.v1.0` |
| `exllamav3-fuzev` | `1.5.2+fuzev.v1.0` |
| `vllm-exl3-plugin` (AGPL-3.0-only) | `0.5.0+fuzev.v1.0` |

## Use

- Each component ships a `VERSION` file and package metadata.
- The [installation guide](install.md) covers both engine paths. Its canonical
  [source-installation checks](install.md#verified-source-installations) link
  the engine configurations and HTTP results.
- vLLM's `VLLM_VERSION_OVERRIDE` supports custom builds. Platform suffixes may
  follow the local version, such as `0.30.0+fuzev.v1.0.cpu`.
- Results and workloads: [benchmarks](benchmarks.md),
  [integration checks](integration.md) and [performance](performance.md).

## Limitations

FuZev v1.0 is an experimental research release. The
[API limitations](api.md#limitations) cover option counts, engine scope and
images. Research methods are in the
[archive](https://github.com/PeisongXiao/fuzev-research-archive).
