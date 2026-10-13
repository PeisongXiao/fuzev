# FuZev Benchmarks

This page reports recorded quality measurements: dataset scores, controls and
uncertainty. To serve decisions, see the [installation guide](install.md) and
[API reference](api.md).

- [Aggregate data](benchmark-results.json): full-precision task scores, controls
  and uncertainty.
- [Source aggregates](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/results/README.md):
  each configuration's benchmark report and summary, TEST controls and processed quality.

## Results

The [headline results](../README.md#results) compare all six configurations and
Kev's full published TEST table.

DEV comparison on the same 3,075 Kev DEV questions:

- `breadth-v1` DEV index, from Kev's development report
  ([Kev 1.0 report](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/breadth-v1-report/report.md)): Jev 53.3 [50.7, 56.3],
  Kev-4B 40.8 [38.2, 44.0].
- DEV question accuracy, from Kev's cards: Jev 75.7% and Kev-9B v2 70.0%
  ([Kev-9B card](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-9b.md#evaluation)),
  Kev-4B 69.6% ([Kev-4B card](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-4b.md#evaluation)),
  Kev-0.8B 59.7% ([Kev-0.8B card](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-0.8b.md#evaluation)).
- FuZev DEV question accuracy: 74.6-75.7% across the six configurations
  ([selection](#selection-and-modality)).

Lowest and trailing results:

- TEST datasets: ChessBench 23.33%, SATA-Bench case-exact 39.33%, RouterBench
  40.00%, cfcolor 52.00%, Humicroedit 56.67%.
- HelpSteer2 selection on the evaluation mix: correctness 42-50%, helpfulness
  40-51%.
- VL-RewardBench preference accuracy trails the backbone control on the full
  held-out split by 20.0 points (Choice) and 12.8 points (Noul). On the
  evaluation mix it varies by up to 19 points across configurations
  (Noul 58-77%, Choice 55-69%).
- [TEST Controls And Datasets](#test-controls-and-datasets) lists where FuZev
  trails Kev-9B v2 and the frozen-LM control.

## Workload And Settings

Each configuration completes 5,075 requests with zero response-validation errors.

| Workload | Records | Requests | Text Requests | Image Requests |
| --- | ---: | ---: | ---: | ---: |
| FuZev evaluation mix | 2,000 | 2,000 | 1,400 | 600 |
| Kev DEV | 1,990 | 3,075 | 3,075 | 0 |
| Total per configuration | 3,990 | 5,075 | 4,475 | 600 |

- **Platform:** NVIDIA GB10, Python 3.12, PyTorch 2.13.0+cu130.
- **Checkpoints:** [Models](models.md#recorded-benchmark-settings) lists
  revisions, KV formats and embedding storage.
- **MTP:** off.
- **Shared head:** raw temperature 1.
- **Bootstrap intervals:** 2,000 whole-record draws within each dataset, seed 0.
- **`breadth-v1` index:** chance-corrected, using the community Decision Index 0.2
  formula. Each dataset's value is (score - chance) / (1 - chance), clipped to [0, 1].
  Values average within each of five areas; the index is 100 times the area mean.

Table abbreviations:

- **N3/N4/N5:** ExLlamaV3 EXL3 3.05/4.05/5.05 bpw.
- **NV/RX:** vLLM NVIDIA/RadixArk NVFP4.
- **VX:** vLLM EXL3 4.05 bpw.

### Selection And Modality

Selection percentages are descriptive; the index uses the official task metrics.
Evaluation-mix selection counts any equally correct option as a hit.

| Configuration | Kev DEV Selection | Evaluation-Mix Selection |
| --- | ---: | ---: |
| ExLlamaV3 EXL3 3.05 bpw | 2319/3,075 (75.41%) | 1521/2,000 (76.05%) |
| ExLlamaV3 EXL3 4.05 bpw | 2299/3,075 (74.76%) | 1568/2,000 (78.40%) |
| ExLlamaV3 EXL3 5.05 bpw | 2314/3,075 (75.25%) | 1545/2,000 (77.25%) |
| vLLM NVIDIA NVFP4 | 2293/3,075 (74.57%) | 1549/2,000 (77.45%) |
| vLLM RadixArk NVFP4 | 2329/3,075 (75.74%) | 1529/2,000 (76.45%) |
| vLLM EXL3 4.05 bpw | 2299/3,075 (74.76%) | 1556/2,000 (77.80%) |

| Workload / Modality | Requests | N3 | N4 | N5 | NV | RX | VX |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Evaluation-mix text | 1400 | 76.57 | 76.93 | 77.21 | 76.86 | 75.93 | 76.29 |
| Evaluation-mix image | 600 | 74.83 | 81.83 | 77.33 | 78.83 | 77.67 | 81.33 |
| Kev DEV text | 3075 | 75.41 | 74.76 | 75.25 | 74.57 | 75.74 | 74.76 |

### Kev DEV Datasets

Scores are percentages. BFCL and SATA-Bench use record-level case-exact scoring;
other datasets use question accuracy.

| Dataset | Units / Requests | N3 | N4 | N5 | NV | RX | VX |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| API-Bank | 150 / 150 | 92.00 | 92.00 | 91.33 | 92.67 | 94.00 | 91.33 |
| BFCL (case-exact) | 150 / 306 | 93.33 | 91.33 | 93.33 | 89.33 | 94.00 | 90.00 |
| BRIGHT | 150 / 150 | 68.67 | 67.33 | 67.33 | 68.67 | 68.67 | 69.33 |
| cfcolor | 150 / 150 | 51.33 | 48.67 | 50.00 | 49.33 | 50.00 | 49.33 |
| ChessBench | 150 / 150 | 32.00 | 30.67 | 32.67 | 28.00 | 28.00 | 26.00 |
| CLINC150 | 150 / 150 | 96.67 | 98.00 | 98.00 | 96.00 | 98.00 | 97.33 |
| ContractNLI | 160 / 160 | 68.13 | 75.00 | 72.50 | 72.50 | 76.88 | 73.13 |
| HellaSwag | 150 / 150 | 88.67 | 88.00 | 89.33 | 85.33 | 86.67 | 86.67 |
| Humicroedit | 150 / 150 | 56.67 | 56.00 | 55.33 | 56.67 | 61.33 | 57.33 |
| MuSR | 150 / 150 | 71.33 | 68.67 | 68.67 | 68.00 | 69.33 | 69.33 |
| RouterBench | 150 / 150 | 37.33 | 32.67 | 34.67 | 32.00 | 41.33 | 37.33 |
| SATA-Bench (case-exact) | 150 / 959 | 50.67 | 49.33 | 52.00 | 49.33 | 48.00 | 50.67 |
| SGD / SGD-X | 150 / 150 | 71.33 | 71.33 | 70.67 | 78.00 | 74.00 | 72.67 |
| ToolRet | 150 / 150 | 63.33 | 61.33 | 63.33 | 63.33 | 63.33 | 63.33 |

BFCL and SATA-Bench also have descriptive question accuracies:

| Dataset | Requests | N3 | N4 | N5 | NV | RX | VX |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BFCL | 306 | 96.73 | 93.79 | 95.75 | 93.79 | 96.73 | 92.81 |
| SATA-Bench | 959 | 85.51 | 85.51 | 85.82 | 84.78 | 84.25 | 85.30 |

## TEST Controls And Datasets

- **Frozen-LM label control:** reads the backbone's label logits without the
  learned head.
- **Feature-only head:** removes the LM-label contribution.
- All three FuZev systems use the same EXL3 4.05-bpw features and TEST population.
  Only this configuration has a TEST read.
- TEST scoring runs offline from extracted features: eight candidate slots,
  8,192-token chunks, FP16 KV. DEV runs call the HTTP server.

| System | Question Accuracy (%) | `breadth-v1` TEST Index | 95% CI |
| --- | ---: | ---: | ---: |
| FuZev head | 73.2 | 46.8 | [44.2, 50.0] |
| Frozen-LM label control | 67.0 | 42.4 | [40.2, 45.7] |
| Feature-only head | 72.3 | 45.3 | [42.5, 48.5] |
| [Kev-0.8B (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-0.8b.md#evaluation) | 58.6 | 23.3 | [21.2, 25.9] |
| [Kev-4B (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-4b.md#evaluation) | 69.0 | 38.0 | [35.5, 41.3] |
| [Kev-9B v2 (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-9b.md#evaluation) | 69.8 | 41.0 | [38.8, 43.9] |
| [Kev-27B (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-27b.md#evaluation) | 75.7 | 52.3 | [49.2, 55.4] |
| [Jev, published by Kev (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) | 75.7 | 54.0 | [51.2, 57.0] |

Kev and Jev intervals come from each Kev model card's Evaluation section.

- Paired head minus frozen LM: +4.4 [2.0, 6.4] index points.
- Paired head minus feature-only: +1.5 [-0.5, +3.7].
- Feature-only scores higher on BRIGHT, cfcolor, BFCL and SATA-Bench.

Dataset scores are percentages under DEV's official metric conventions. The
Kev-9B v2 column is `systems["Kev-9B"].datasets` from
[Kev's TEST report](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json).

| Dataset | Units | FuZev | Frozen LM | Feature Only | Kev-9B v2 |
| --- | ---: | ---: | ---: | ---: | ---: |
| API-Bank | 150 | 94.67 | 92.00 | 94.00 | 95.33 |
| BFCL (case-exact) | 150 | 87.33 | 81.33 | 88.00 | 91.33 |
| BRIGHT | 150 | 68.67 | 67.33 | 72.67 | 60.00 |
| cfcolor | 150 | 52.00 | 50.00 | 58.00 | 51.33 |
| ChessBench | 150 | 23.33 | 19.33 | 18.67 | 20.67 |
| CLINC150 | 150 | 95.33 | 92.67 | 85.33 | 81.33 |
| ContractNLI | 160 | 67.50 | 71.88 | 66.88 | 71.88 |
| HellaSwag | 150 | 82.67 | 81.33 | 79.33 | 75.33 |
| Humicroedit | 150 | 56.67 | 54.67 | 54.00 | 47.33 |
| MuSR | 150 | 65.33 | 51.33 | 59.33 | 61.33 |
| RouterBench | 150 | 40.00 | 36.00 | 35.33 | 18.00 |
| SATA-Bench (case-exact) | 150 | 39.33 | 20.00 | 41.33 | 33.33 |
| SGD / SGD-X | 150 | 65.33 | 68.67 | 64.67 | 65.33 |
| ToolRet | 150 | 61.33 | 59.33 | 61.33 | 55.33 |

- FuZev trails Kev-9B v2 on ContractNLI, BFCL and API-Bank, and ties on SGD / SGD-X.
- FuZev trails its frozen-LM control on ContractNLI and SGD / SGD-X.

On the [headline comparison](../README.md#breadth-v1-test), FuZev is above
Kev-0.8B, Kev-4B and Kev-9B v2 and below Kev-27B and Jev. This comparison uses
unpaired marginal intervals, and Kev reports Jev from a separate read of the same
TEST items. Training differs (each Kev card describes it):

- FuZev trains only a head on a frozen backbone.
- Kev-0.8B/4B/9B fine-tune backbone LoRA adapters and pointer heads.
- Kev-27B fine-tunes its backbone weights.

## All 15 Evaluation-Mix Tracks

Unweighted evaluation-mix selection accuracy in percent. Each dataset, method,
modality and subtype stays separate.

| Dataset / Method / Modality / Subtype | Requests | N3 | N4 | N5 | NV | RX | VX |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BoolQ / Noul / text / boolean truth | 300 | 90.67 | 91.00 | 90.00 | 91.00 | 91.33 | 90.33 |
| HelpSteer2 / Choice / text / main | 100 | 85.00 | 82.00 | 81.00 | 85.00 | 86.00 | 84.00 |
| HelpSteer2 / Score / text / coherence | 100 | 71.00 | 70.00 | 73.00 | 68.00 | 70.00 | 71.00 |
| HelpSteer2 / Score / text / complexity | 100 | 61.00 | 67.00 | 64.00 | 67.00 | 61.00 | 65.00 |
| HelpSteer2 / Score / text / correctness | 100 | 50.00 | 48.00 | 48.00 | 42.00 | 45.00 | 44.00 |
| HelpSteer2 / Score / text / helpfulness | 100 | 41.00 | 46.00 | 51.00 | 48.00 | 40.00 | 45.00 |
| HelpSteer2 / Score / text / verbosity | 100 | 67.00 | 71.00 | 71.00 | 67.00 | 63.00 | 69.00 |
| HelpSteer3 / Choice / text / main | 200 | 80.50 | 79.50 | 80.00 | 80.50 | 79.00 | 79.50 |
| MM-RLHF / Choice / image / best response | 150 | 78.00 | 79.33 | 80.67 | 78.67 | 74.00 | 79.33 |
| MM-RLHF / Choice / image / pairwise preference | 250 | 87.60 | 90.40 | 90.00 | 90.40 | 91.20 | 90.80 |
| QASPER / Noul / text / answerability | 100 | 90.00 | 91.00 | 89.00 | 90.00 | 91.00 | 90.00 |
| QASPER / Noul / text / evidence | 150 | 87.33 | 84.67 | 87.33 | 88.00 | 88.00 | 87.33 |
| QASPER / Noul / text / yes no | 50 | 86.00 | 86.00 | 86.00 | 86.00 | 86.00 | 78.00 |
| VL-RewardBench / Choice / image / official heldout preference | 100 | 55.00 | 69.00 | 60.00 | 66.00 | 67.00 | 69.00 |
| VL-RewardBench / Noul / image / pairwise preference yes no | 100 | 58.00 | 77.00 | 58.00 | 63.00 | 60.00 | 73.00 |

### Conditional Weighted Results

Each request's weight is the inverse of its subset-inclusion probability times
its source sampling weight. Values are point estimates, without intervals, for
API-eligible represented strata
([evaluation method](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/research-evaluation.md)).

| Dataset / Method / Modality / Subtype | Requests | N3 | N4 | N5 | NV | RX | VX |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BoolQ / Noul / text / boolean truth | 300 | 90.66 | 90.99 | 89.99 | 90.99 | 91.32 | 90.32 |
| HelpSteer2 / Choice / text / main | 100 | 84.90 | 81.88 | 80.88 | 84.90 | 85.91 | 83.90 |
| HelpSteer2 / Score / text / coherence | 100 | 70.56 | 69.59 | 72.63 | 67.57 | 69.62 | 70.56 |
| HelpSteer2 / Score / text / complexity | 100 | 60.80 | 66.71 | 63.78 | 66.76 | 60.79 | 64.77 |
| HelpSteer2 / Score / text / correctness | 100 | 50.06 | 48.03 | 48.13 | 42.17 | 45.17 | 44.10 |
| HelpSteer2 / Score / text / helpfulness | 100 | 41.04 | 45.99 | 51.10 | 48.01 | 40.13 | 45.01 |
| HelpSteer2 / Score / text / verbosity | 100 | 67.03 | 71.13 | 71.13 | 67.10 | 62.98 | 69.04 |
| HelpSteer3 / Choice / text / main | 200 | 80.48 | 79.48 | 79.98 | 80.48 | 78.98 | 79.48 |
| MM-RLHF / Choice / image / best response | 150 | 78.22 | 79.48 | 80.73 | 78.93 | 74.25 | 79.64 |
| MM-RLHF / Choice / image / pairwise preference | 250 | 87.64 | 90.44 | 90.04 | 90.44 | 91.23 | 90.83 |
| QASPER / Noul / text / answerability | 100 | 90.24 | 91.24 | 89.24 | 90.24 | 91.24 | 90.24 |
| QASPER / Noul / text / evidence | 150 | 93.02 | 89.82 | 95.10 | 96.00 | 93.79 | 94.79 |
| QASPER / Noul / text / yes no | 50 | 85.92 | 85.82 | 85.82 | 85.97 | 85.87 | 77.77 |
| VL-RewardBench / Choice / image / official heldout preference | 100 | 54.85 | 68.90 | 59.87 | 65.89 | 66.89 | 68.90 |
| VL-RewardBench / Noul / image / pairwise preference yes no | 100 | 57.86 | 76.93 | 57.86 | 62.88 | 59.87 | 73.07 |

- QASPER evidence has effective sample size 53.71 across 150 requests. Its
  largest weight, 2,900.92, is about 4.05% of the total weight 71,661.04.
- Both VL-RewardBench methods use the same 100 examples.

### Score Error

HelpSteer2 expected-rating mean absolute error; lower is better. Each subtype
has 100 requests per configuration.

| HelpSteer2 Subtype | N3 | N4 | N5 | NV | RX | VX |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| coherence | 0.4929 | 0.4939 | 0.5027 | 0.5456 | 0.5248 | 0.4976 |
| complexity | 0.4527 | 0.4335 | 0.4462 | 0.4194 | 0.4478 | 0.4259 |
| correctness | 0.9191 | 0.9210 | 0.9236 | 0.9334 | 0.9334 | 0.9349 |
| helpfulness | 0.8250 | 0.8084 | 0.7846 | 0.8441 | 0.8026 | 0.7823 |
| verbosity | 0.4225 | 0.3939 | 0.4063 | 0.3956 | 0.4184 | 0.4048 |

## Full Held-Out Visual Comparison

The full processed TEST split has 25,599 records across 15 tracks: held-out
training-source records plus the external QASPER and VL-RewardBench datasets.
Both VL-RewardBench methods use all 1,247 examples and 1,194 grouped units, with
features from the EXL3 4.05-bpw backbone.

| Method | Records | Grouped Units | Head (%) | Frozen LM (%) | Paired Difference (pp) [95% CI] |
| --- | ---: | ---: | ---: | ---: | ---: |
| Choice preference | 1247 | 1194 | 68.97 | 89.01 | -20.05 [-22.49, -17.54] |
| Noul relative preference | 1247 | 1194 | 69.85 | 82.60 | -12.75 [-15.15, -10.38] |

- For both methods, head selection accuracy is the same raw, calibrated, and
  when counting equally correct options as hits.
- Intervals use 2,000 paired group-bootstrap draws, seed 17.
- The 100-example evaluation-mix subset above samples this held-out set.

## Repeat Probes

Each probe repeats a synthetic two-option decision immediately and after a
different request. Delta columns compare against the first response. Spread is
the largest difference between any two of the three responses.

| Configuration | Context Tokens | Immediate Repeat | After Other Request | Spread | 2e-6 Tolerance |
| --- | --- | ---: | ---: | ---: | --- |
| ExLlamaV3 EXL3 3.05 / 4.05 / 5.05 bpw | 2,048 / 8,192 / 16,384 / 32,768 | 0 (identical) | 0 (identical) | 0 | Pass |
| vLLM NVIDIA NVFP4 | 2,048 | 0.028949 | 0.001992 | 0.0309 | Fail |
| vLLM RadixArk NVFP4 | 2,048 | 0.061121 | 0.190495 | 0.2516 | Fail |
| vLLM EXL3 4.05 bpw | 2,048 | 0.006018 | 0.011897 | 0.0179 | Fail |
| vLLM NVIDIA NVFP4, later trace | 2,048 | 0.104289 | 0.009708 | 0.1043 | Fail |

The [later NVIDIA trace](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/results/repeat/nvidia-later-trace.json)
uses the NVIDIA NVFP4 benchmark run's vLLM engine build with an instrumented
evaluation server. It records a maximum change of 0.10429 from the first response
and 46-48% relative L2 drift in the query features.

Each ExLlamaV3 repeat runs within one server. ExLlamaV3 probabilities can differ
slightly:

- between server starts, because Triton selects kernel configurations by timing
  when it first compiles them;
- with which prefix pages are already cached.

### Run-To-Run Variation

Three runs of the vLLM NVIDIA NVFP4 configuration share 670 identical
evaluation-mix requests. Two runs are partial; their completed responses supply
this comparison ([archive records](https://github.com/PeisongXiao/fuzev-research-archive/blob/main/docs/results/README.md#six-quality-panels)).

- Between any two runs, 14.6-14.9% of selections differ and probabilities differ
  by up to 0.62.
- Per-run accuracy on those requests is 73.4%, 74.8% and 75.2%.

No other configuration has multiple runs with completed decisions. Sampling
intervals and repeat probes describe different sources of uncertainty.

## Limitations

- Results describe pre-release builds of the archived research evaluation
  servers with their recorded settings.
- The published serving implementations have no end-to-end quality benchmark.
- The three vLLM quality runs predate two vLLM-fork changes that affect decision
  features: CUDA-graph replay of per-layer embeddings, and KV-cache slot mapping.
  These runs were not repeated on the published build.
- The ExLlamaV3 quality runs use a pre-release fork build. Its extension predates
  the released CUDA-graph kernel-argument fix, and its optimized prefill casts
  gated-delta-network operands to FP16. These runs were not repeated on the released build.
- vLLM repeat variation can exceed differences between configuration indices.
- Visual preference quality regresses against the frozen-LM control.
- Published ExLlamaV3 image requests have CPU coverage only.

See also:

- [API limitations](api.md#limitations): option counts and supported engine settings.
- [Performance](performance.md): short workloads with timing ranges.
- [Integration](integration.md): generation and MTP checks.
