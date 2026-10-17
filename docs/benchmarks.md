# FuZev Benchmarks

This page reports recorded quality measurements: dataset scores, controls and
uncertainty, plus decode speed. To serve decisions, see the
[installation guide](install.md) and [API reference](api.md).

- [Aggregate data](https://github.com/PeisongXiao/fuzev-research-archive/tree/v1.1/docs/results/comparison/v1.1):
  full-precision task scores, controls and uncertainty.
- [Source aggregates](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/README.md):
  each configuration's benchmark report and summary, TEST controls and processed quality.

## Results

The [headline results](../README.md#results) compare FuZev v1.0 and v1.1,
all six configurations and Kev's full published TEST table.

Decision results were measured on the v1.1 release candidate before the
decision-root scheduling fix; prefill and decode do not use that path.

DEV comparison on the same 3,075 Kev DEV questions:

- `breadth-v1` DEV index, from Kev's development report
  ([Kev 1.0 report](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/breadth-v1-report/report.md)): Jev 53.3 [50.7, 56.3],
  Kev-4B 40.8 [38.2, 44.0].
- DEV question accuracy, from Kev's cards: Jev 75.7% and Kev-9B v2 70.0%
  ([Kev-9B card](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-9b.md#evaluation)),
  Kev-4B 69.6% ([Kev-4B card](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-4b.md#evaluation)),
  Kev-0.8B 59.7% ([Kev-0.8B card](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-0.8b.md#evaluation)).
- FuZev DEV question accuracy: 74.37% ([DEV](#dev)); the FuZev 1.0 head scores
  74.6-75.7% across the six study configurations
  ([selection](#selection-and-modality)).

Kev and Jev: results published by the repo owner.

Lowest and trailing results:

- TEST datasets: ChessBench 21.33%, RouterBench 32.00%, SATA-Bench case-exact
  41.33%, cfcolor 54.67%, Humicroedit 56.00%.
- HelpSteer2 selection on the FuZev 1.0 head study's evaluation mix:
  correctness 42-50%, helpfulness 40-51%.
- Experimental image decisions: MM-RLHF best response 56.67%, and full
  VL-RewardBench Choice 72.33% against the frozen-LM label control's 84.96%
  ([images](#experimental-image-decisions)). In this conversion the preferred
  response is first in 1,226 of 1,247 VL-RewardBench items (98.3%), so
  accuracy mostly reflects how often a system answers Yes or picks the first
  response. On the 21 items where the second response is preferred, the head
  answers 16 and the control 13 correctly.
- FuZev trails Kev-9B v2 on API-Bank, BFCL and ContractNLI and ties on MuSR
  ([TEST](#test)).

### Setup

Both FuZev configurations use turboderp's EXL3 4.05 bpw checkpoint
(revision `55a732e0`), batch 4 and MTP 3. The v1.1 server uses Q8 KV cache;
the v1.0 record does not state its KV format. Each Kev record's
questions are sent together in one `/v1/systemone` request, one pass per
record; missing responses count as wrong.

| Configuration | Server And Head | Context / Cache Tokens | Runtime Prefill Chunk |
| --- | --- | --- | ---: |
| FuZev v1.1 | `fuzev-native` 1.1.0, exllamav3-fuzev 1.6.1+fuzev.v1.1, head 1.1.0, calibrated | 98,304 / 98,304 | 32,768 |
| FuZev v1.0 | TabbyAPI-based server on exllamav3-fuzev 1.5.2+fuzev.v1.0, head 1.0.0, raw | 98,304-token context | 8,192 |

FuZev trains a 2.9M-parameter head on a frozen backbone. Kev-0.8B/4B/9B
fine-tune LoRA adapters and pointer heads on dense Qwen3.5 bases;
Kev-27B fine-tunes the backbone; Jev is hosted. The server default is raw;
FuZev v1.1 measurements select calibrated mode.

[Comparison method](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/v1.0-v1.1-comparison.md),
[curated records](https://github.com/PeisongXiao/fuzev-research-archive/tree/v1.1/docs/results/comparison/v1.1)
and [plot script](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/tools/make_plots.py).

### Text Decisions

Suite: `breadth-v1` from Kev 1.0. DEV has 3,075 questions; TEST has 3,089.
The index chance-corrects each dataset score, clips it to [0, 1], averages
within each of five areas, then takes 100 times the area mean. BFCL and
SATA-Bench use record-level case-exact scoring; other dataset rows use
question accuracy. Descriptive question accuracies for BFCL and SATA-Bench
are also shown. Scores are percentages, except the 0-100 index.

#### DEV

Kev and Jev: results published by the repo owner.

| Dataset / Metric (%) | FuZev v1.0 | FuZev v1.1 | [Kev-0.8B](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-08b-breadth/rows.json) | [Kev-4B](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/breadth-v1-report/report.json) | [Kev-9B v2](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-9bnew-breadth/rows.json) | [Kev-27B v2](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/r23-27b-k-w85-breadth/rows.json) | [Jev](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/breadth-v1-report/report.json) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| API-Bank | 92.00 | 91.33 | 74.00 | 88.00 | 89.33 | 91.33 | 94.67 |
| BFCL (case-exact) | 92.00 | 92.00 | 74.67 | 94.00 | 94.67 | 96.00 | 98.00 |
| BRIGHT | 69.33 | 68.00 | 36.00 | 56.00 | 54.67 | 66.67 | 69.33 |
| cfcolor | 49.33 | 58.00 | 50.67 | 50.00 | 48.00 | 54.00 | 54.67 |
| ChessBench | 30.67 | 28.00 | 24.00 | 18.00 | 23.33 | 27.33 | 31.33 |
| CLINC150 | 97.33 | 88.67 | 77.33 | 88.00 | 83.33 | 94.67 | 97.33 |
| ContractNLI | 71.25 | 70.00 | 41.25 | 63.75 | 74.38 | 80.63 | 79.38 |
| HellaSwag | 88.67 | 91.33 | 50.67 | 78.00 | 81.33 | 88.67 | 89.33 |
| Humicroedit | 56.00 | 59.33 | 44.67 | 57.33 | 56.00 | 59.33 | 58.00 |
| MuSR | 73.33 | 71.33 | 46.67 | 53.33 | 56.67 | 62.67 | 69.33 |
| RouterBench | 34.67 | 30.67 | 26.67 | 34.00 | 18.00 | 33.33 | 28.67 |
| SATA-Bench (case-exact) | 48.67 | 47.33 | 27.33 | 35.33 | 38.00 | 48.67 | 43.33 |
| SGD / SGD-X | 71.33 | 66.00 | 56.67 | 59.33 | 66.00 | 72.67 | 79.33 |
| ToolRet | 61.33 | 62.00 | 49.33 | 62.00 | 66.00 | 68.00 | 63.33 |
| BFCL question accuracy | 94.12 | 94.77 | 80.07 | 96.41 | 96.73 | 98.04 | 99.02 |
| SATA-Bench question accuracy | 85.09 | 84.78 | 75.08 | 80.92 | 80.50 | 85.71 | 83.00 |
| Overall question accuracy | 74.93 | 74.37 | 59.71 | 69.56 | 69.95 | 75.74 | 75.74 |
| breadth-v1 index [95% CI] | 50.26 [47.83, 53.30] | 50.62 [47.53, 53.66] | 25.35 [23.11, 28.21] | 40.78 [38.25, 44.00] | 43.31 [40.86, 46.27] | 52.03 [49.42, 54.93] | 53.35 [50.67, 56.32] |

#### TEST

Kev and Jev: results published by the repo owner.

| Dataset / Metric (%) | FuZev v1.0 | FuZev v1.1 | [Kev-0.8B](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Kev-4B](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Kev-9B v2](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Kev-27B v2](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Jev](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/r23-breadth-report/report.json) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| API-Bank | 93.33 | 90.00 | 76.67 | 89.33 | 95.33 | 94.67 | 92.67 |
| BFCL (case-exact) | 86.67 | 89.33 | 70.67 | 90.00 | 91.33 | 93.33 | 91.33 |
| BRIGHT | 64.67 | 69.33 | 35.33 | 52.67 | 60.00 | 68.00 | 72.00 |
| cfcolor | 52.67 | 54.67 | 46.67 | 50.67 | 51.33 | 65.33 | 66.00 |
| ChessBench | 20.67 | 21.33 | 16.67 | 14.00 | 20.67 | 24.67 | 25.33 |
| CLINC150 | 95.33 | 86.67 | 74.00 | 79.33 | 81.33 | 91.33 | 95.33 |
| ContractNLI | 63.75 | 60.63 | 38.13 | 55.00 | 71.88 | 76.25 | 80.63 |
| HellaSwag | 82.00 | 88.00 | 51.33 | 70.67 | 75.33 | 87.33 | 88.00 |
| Humicroedit | 57.33 | 56.00 | 44.00 | 54.00 | 47.33 | 56.67 | 59.33 |
| MuSR | 64.00 | 61.33 | 46.67 | 60.00 | 61.33 | 67.33 | 67.33 |
| RouterBench | 39.33 | 32.00 | 34.00 | 34.67 | 18.00 | 30.67 | 38.00 |
| SATA-Bench (case-exact) | 42.00 | 41.33 | 18.67 | 34.67 | 33.33 | 42.00 | 33.33 |
| SGD / SGD-X | 63.33 | 66.00 | 48.67 | 67.33 | 65.33 | 72.00 | 73.33 |
| ToolRet | 61.33 | 60.00 | 53.33 | 58.67 | 55.33 | 64.00 | 62.67 |
| BFCL question accuracy | 92.46 | 93.77 | 81.97 | 94.43 | not reported | 95.74 | 94.75 |
| SATA-Bench question accuracy | 84.09 | 84.09 | 72.79 | 82.96 | not reported | 86.34 | 83.06 |
| Overall question accuracy | 72.64 | 72.19 | 58.63 | 68.99 | 69.83 | 75.69 | 75.66 |
| breadth-v1 index [95% CI] | 45.57 [42.95, 48.67] | 45.24 [42.46, 48.58] | 23.34 [21.21, 25.91] | 38.03 [35.49, 41.26] | 40.98 [38.82, 43.90] | 52.29 [49.22, 55.35] | 54.05 [51.15, 57.01] |

#### TEST By Area

Area scores are 100 times the mean chance-corrected dataset skill.
The index is the mean of the five area scores.

Kev and Jev: results published by the repo owner.

| TEST Area (Chance-Corrected Score) | FuZev v1.0 | FuZev v1.1 | [Kev-0.8B](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Kev-4B](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Kev-9B v2](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Kev-27B v2](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json) | [Jev](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/r23-breadth-report/report.json) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Knowledge & Reasoning | 30.48 | 29.07 | 11.64 | 23.28 | 26.10 | 33.82 | 31.13 |
| Language Understanding | 60.81 | 62.47 | 21.15 | 46.69 | 62.46 | 73.74 | 77.47 |
| Retrieval & Classification | 64.68 | 64.79 | 37.01 | 55.81 | 58.58 | 69.05 | 72.88 |
| Tools & Automation | 61.87 | 59.19 | 46.90 | 59.67 | 56.44 | 62.86 | 63.43 |
| Arts & Human Taste | 10.00 | 10.67 | 0.00 | 4.67 | 1.33 | 22.00 | 25.33 |

#### Paired Differences

![Per-dataset score change from FuZev v1.0 to v1.1 on DEV and TEST with paired 95% bootstrap intervals](images/text-deltas.png)

Each bar is v1.1 minus v1.0, in dataset-score percentage points. Whiskers
are paired 95% intervals from 2,000 whole-record resamples within each
dataset, seed 0. The full [paired DEV](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-dev-v1.1-vs-v1.0.json)
and [paired TEST](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-test-v1.1-vs-v1.0.json)
records hold every delta and interval.

| Index Difference | Point Estimate | Paired 95% CI |
| --- | ---: | --- |
| FuZev v1.1 minus v1.0, DEV | +0.36 | [-2.84, +2.89] |
| FuZev v1.1 minus v1.0, TEST | -0.33 | [-3.35, +2.51] |
| FuZev v1.1 minus Kev-9B v2, DEV | +7.31 | [+3.75, +10.30] |

Text decision quality is unchanged from v1.0 overall within the CI;
CLINC150 drops 8.67 points on both splits. Kev publishes no TEST
predictions, so TEST has no paired comparison. The DEV difference to
Kev-9B v2 uses its published rows
([paired record](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-dev-paired.json)).

#### Method And Variability

Each FuZev configuration is one served run. Bootstrap intervals use 2,000
whole-record draws within each dataset, seed 0. DEV indices for Kev-0.8B,
Kev-9B v2 and Kev-27B v2 are computed from the repo owner's published rows
with the `breadth-v1` formula. Kev and Jev use Kev 1.0's own harness;
the column links identify each primary source. The
[curated reference record](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-published.json)
holds the source URLs, dataset values and uncertainty.

The same v1.0 head and 4.05 bpw checkpoint differ from the v1.0 study's
DEV run by up to 4.67 points on one dataset (MuSR) and 0.16 points in
overall question accuracy (2,304 vs 2,299 of 3,075 questions). Sampling
intervals describe dataset uncertainty; these setup-dependent differences
also affect comparisons.

Served source records: [v1.0 DEV](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-dev-v1.0.json),
[v1.1 DEV](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-dev-v1.1.json),
[v1.0 TEST](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-test-v1.0.json),
[v1.1 TEST](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/kev-test-v1.1.json).

### Experimental Image Decisions

Image input is an experimental extension (run at your own risk). The
comparison uses the same 4.05 bpw backbone. Cells below are **accuracy (%) /
NLL / ECE**. Accuracy counts any equally best option. NLL uses natural
logarithms and is cross-entropy against the target distribution; tied-gold
records split the target equally (47 of 150 MM-RLHF best-response records).
ECE is expected calibration error.

FuZev v1.1 runs calibrated: image Choice with 2 options uses its fitted
temperature, T = 0.514; image Noul and image Choice with 4 or 5 options use
raw probabilities. The frozen-LM label control reads the v1.1 backbone's
label logits without the head. The v1.0 head uses raw probabilities in
both serving builds.

VL-RewardBench: in this conversion the preferred response is first in
1,226 of 1,247 items (98.3%), so accuracy mostly reflects how often a
system answers Yes or picks the first response. The mix contains 98
first-preferred items of 100. The second-preferred subset results follow
the table.

| Workload (Requests) | FuZev v1.0 | FuZev v1.1 + Head 1.0.0 | FuZev v1.1 | Frozen-LM Label Control |
| --- | ---: | ---: | ---: | ---: |
| MM-RLHF best response (150) | 73.33 / 0.920 / 0.091 | 78.67 / 0.905 / 0.075 | 56.67 / 1.285 / 0.093 | 57.33 / 1.626 / 0.230 |
| MM-RLHF pairwise (250) | 88.80 / 0.465 / 0.045 | 88.40 / 0.466 / 0.055 | 80.80 / 0.804 / 0.166 | 77.60 / 0.849 / 0.201 |
| VL-RewardBench mix Choice (100) | 72.00 / 0.617 / 0.108 | 72.00 / 0.636 / 0.108 | 73.00 / 0.800 / 0.153 | 83.00 / 0.412 / 0.086 |
| VL-RewardBench mix Noul (100) | 75.00 / 0.573 / 0.088 | 77.00 / 0.516 / 0.058 | 89.00 / 0.279 / 0.055 | 80.00 / 0.426 / 0.113 |
| VL-RewardBench full Choice (1,247) | 69.59 / 0.612 / 0.049 | 70.47 / 0.605 / 0.038 | 72.33 / 0.686 / 0.106 | 84.96 / 0.364 / 0.049 |
| VL-RewardBench full Noul (1,247) | 72.81 / 0.528 / 0.035 | 75.14 / 0.506 / 0.019 | 92.20 / 0.228 / 0.056 | 81.82 / 0.400 / 0.029 |
| SugarCrepe attribute-caption (192 views) | 98.44 / 0.141 / 0.101 | 98.44 / 0.138 / 0.105 | 98.44 / 0.046 / 0.021 | 96.88 / 0.069 / 0.053 |

Each configuration initially returns HTTP 413 for 23 requests; 15 succeed
on a lossless WebP retry and 8 remain unscored. Four requests per full
VL-RewardBench track exceed the default 8 MiB request-body limit
(`--max-request-bytes`); 1,243 of 1,247 are scored on each track.
All other rows above score the full stated population.

Correct answers on items where the second response is preferred:

| Second-Preferred Subset | FuZev v1.0 | FuZev v1.1 + Head 1.0.0 | FuZev v1.1 | Frozen-LM Label Control |
| --- | ---: | ---: | ---: | ---: |
| VL-RewardBench mix Choice | 2/2 | 2/2 | 2/2 | 2/2 |
| VL-RewardBench mix Noul | 1/2 | 1/2 | 0/2 | 1/2 |
| VL-RewardBench full Choice | 13/21 | 14/21 | 16/21 | 13/21 |
| VL-RewardBench full Noul | 14/21 | 15/21 | 7/21 | 10/21 |

![Experimental image decision accuracy for four systems on the 600-request image mix, full VL-RewardBench tracks and SugarCrepe held-out views](images/image-decisions.png)

The mix has **600 requests** (MM-RLHF best response 150, pairwise 250,
VL-RewardBench Choice 100 and Noul 100). VL-RewardBench preferred-first
balance is 98.3% for the full set and 98% for the mix; the second-preferred
counts above apply to this chart. Accuracy on these converted requests
reflects answer order as well as the chosen content.

SugarCrepe uses `replace_att` pairs on COCO 2017 validation images. Its fit
and development views set the image residual strength and temperature.
Held-out evaluation has 192 views in 96 image groups: 189/192 correct for
head 1.1.0 versus 186/192 for the frozen-LM label control, a paired gain of
+1.6 pp with 95% image-unit interval [0, +3.6]. On the held-out feature
assessment, raw NLL/log 2 is 0.114 vs 0.102 and ECE 0.054 vs 0.053;
calibrated mode improves both. [Image setting and held-out metrics](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/v1.1-head-results.md#visual-eval).

Sources: [FuZev v1.0](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/vision-v1.0.json),
[FuZev v1.1 with head 1.0.0](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/vision-v1.1-v1-head.json),
[FuZev v1.1 and frozen-LM label control](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/comparison/v1.1/vision-v1.1.json).

### Decode

![Decode throughput with MTP 3 and MTP off across six ExLlamaV3 builds and four prompt lengths](images/decode.png)

Measured on one NVIDIA GB10 (128 GB unified memory); Python 3.12.3,
PyTorch 2.13.0+cu130, Triton 3.7.1, FlashInfer 0.6.18.post1 (JIT cache
0.6.18.post1+cu130); engines exllamav3-fuzev 1.6.1+fuzev.v1.1,
exllamav3-fuzev 1.5.2+fuzev.v1.0, ExLlamaV3 1.6.2 and ExLlamaV3 master
(058b8630); batch 1, Q8 KV cache, 32,768-token chunks, 98,304-token
context and cache, FP32 recurrent state unless the row says default FP16.
MTP component loaded; prefill uses MTP off, decode uses MTP 3 or off.

Every measured run generates 1,024 tokens. Sampling: temperature 1.0,
top-p 0.95, top-k 20, seed 0. Each cell follows one warmup and reports
two repetitions at different prompt offsets. Decode rate counts tokens
after the first accepted token; TTFT is host time to the first token.
The MTP draft component remains loaded when MTP is off.

#### MTP 3

| Prompt Tokens | Build | Decode tok/s, Reps 1 / 2 | TTFT Seconds, Reps 1 / 2 | Accepted/Proposed Drafts, Reps 1 ; 2 |
| --- | ---: | ---: | ---: | ---: |
| 2,048 | FuZev v1.0 (FP32) | 42.4 / 36.1 | 2.29 / 2.61 | 646/1134 ; 586/1314 |
| 2,048 | FuZev v1.1 (FP32) | 43.1 / 36.6 | 1.61 / 1.67 | 644/1140 ; 571/1359 |
| 2,048 | ExLlamaV3 1.6.2 (FP32) | 43.1 / 36.1 | 2.52 / 2.69 | 658/1098 ; 584/1320 |
| 2,048 | ExLlamaV3 1.6.2 (default FP16) | 51.7 / 39.7 | 2.24 / 2.22 | 710/942 ; 606/1254 |
| 2,048 | ExLlamaV3 master (FP32) | 43.0 / 36.1 | 2.58 / 2.76 | 658/1098 ; 584/1320 |
| 2,048 | ExLlamaV3 master (default FP16) | 51.1 / 39.0 | 2.06 / 2.17 | 710/942 ; 606/1254 |
| 4,096 | FuZev v1.0 (FP32) | 45.7 / 35.1 | 2.85 / 3.06 | 688/1008 ; 577/1341 |
| 4,096 | FuZev v1.1 (FP32) | 40.7 / 35.6 | 2.66 / 2.68 | 622/1206 ; 559/1395 |
| 4,096 | ExLlamaV3 1.6.2 (FP32) | 48.8 / 34.1 | 3.48 / 3.53 | 708/948 ; 554/1410 |
| 4,096 | ExLlamaV3 1.6.2 (default FP16) | 43.4 / 37.4 | 3.43 / 3.40 | 650/1122 ; 582/1326 |
| 4,096 | ExLlamaV3 master (FP32) | 48.9 / 34.5 | 3.74 / 3.76 | 708/948 ; 554/1410 |
| 4,096 | ExLlamaV3 master (default FP16) | 43.1 / 37.3 | 3.44 / 3.45 | 650/1122 ; 582/1326 |
| 8,192 | FuZev v1.0 (FP32) | 50.9 / 34.7 | 5.14 / 5.16 | 722/906 ; 563/1383 |
| 8,192 | FuZev v1.1 (FP32) | 49.0 / 40.6 | 4.68 / 4.69 | 693/993 ; 619/1215 |
| 8,192 | ExLlamaV3 1.6.2 (FP32) | 47.2 / 38.1 | 6.41 / 6.38 | 691/999 ; 601/1269 |
| 8,192 | ExLlamaV3 1.6.2 (default FP16) | 48.4 / 35.3 | 6.09 / 6.15 | 688/1008 ; 559/1395 |
| 8,192 | ExLlamaV3 master (FP32) | 47.8 / 38.4 | 6.68 / 6.52 | 691/999 ; 601/1269 |
| 8,192 | ExLlamaV3 master (default FP16) | 47.2 / 34.8 | 6.32 / 6.31 | 688/1008 ; 559/1395 |
| 32,768 | FuZev v1.0 (FP32) | 36.9 / 40.6 | 15.83 / 15.66 | 603/1263 ; 643/1143 |
| 32,768 | FuZev v1.1 (FP32) | 39.6 / 48.5 | 15.15 / 15.19 | 614/1230 ; 691/999 |
| 32,768 | ExLlamaV3 1.6.2 (FP32) | 37.3 / 44.2 | 22.12 / 22.03 | 605/1257 ; 669/1065 |
| 32,768 | ExLlamaV3 1.6.2 (default FP16) | 40.4 / 51.5 | 21.45 / 21.50 | 621/1209 ; 709/945 |
| 32,768 | ExLlamaV3 master (FP32) | 37.2 / 43.6 | 21.87 / 21.89 | 605/1257 ; 669/1065 |
| 32,768 | ExLlamaV3 master (default FP16) | 40.6 / 51.1 | 21.62 / 21.62 | 621/1209 ; 709/945 |

Measured on one NVIDIA GB10 (128 GB unified memory); Python 3.12.3,
PyTorch 2.13.0+cu130, Triton 3.7.1, FlashInfer 0.6.18.post1 (JIT cache
0.6.18.post1+cu130); engines exllamav3-fuzev 1.6.1+fuzev.v1.1,
exllamav3-fuzev 1.5.2+fuzev.v1.0, ExLlamaV3 1.6.2 and ExLlamaV3 master
(058b8630); batch 1, Q8 KV cache, 32,768-token chunks, 98,304-token
context and cache, FP32 recurrent state unless the row says default FP16.
MTP component loaded; prefill uses MTP off, decode uses MTP 3 or off.

#### MTP Off

| Prompt Tokens | Build | Decode tok/s, Reps 1 / 2 | TTFT Seconds, Reps 1 / 2 | Accepted/Proposed Drafts, Reps 1 ; 2 |
| --- | ---: | ---: | ---: | ---: |
| 2,048 | FuZev v1.0 (FP32) | 30.4 / 29.9 | 1.53 / 1.74 | 0/0 ; 0/0 |
| 2,048 | FuZev v1.1 (FP32) | 30.2 / 30.0 | 1.54 / 1.58 | 0/0 ; 0/0 |
| 2,048 | ExLlamaV3 1.6.2 (FP32) | 28.9 / 28.5 | 2.47 / 2.37 | 0/0 ; 0/0 |
| 2,048 | ExLlamaV3 1.6.2 (default FP16) | 30.3 / 29.8 | 1.99 / 2.02 | 0/0 ; 0/0 |
| 2,048 | ExLlamaV3 master (FP32) | 29.1 / 29.0 | 2.77 / 2.43 | 0/0 ; 0/0 |
| 2,048 | ExLlamaV3 master (default FP16) | 29.8 / 29.3 | 2.10 / 2.27 | 0/0 ; 0/0 |
| 4,096 | FuZev v1.0 (FP32) | 29.8 / 29.4 | 2.57 / 2.59 | 0/0 ; 0/0 |
| 4,096 | FuZev v1.1 (FP32) | 30.0 / 29.8 | 2.56 / 2.57 | 0/0 ; 0/0 |
| 4,096 | ExLlamaV3 1.6.2 (FP32) | 28.8 / 28.7 | 3.25 / 3.46 | 0/0 ; 0/0 |
| 4,096 | ExLlamaV3 1.6.2 (default FP16) | 29.9 / 29.9 | 3.47 / 3.23 | 0/0 ; 0/0 |
| 4,096 | ExLlamaV3 master (FP32) | 29.2 / 29.0 | 3.24 / 3.26 | 0/0 ; 0/0 |
| 4,096 | ExLlamaV3 master (default FP16) | 29.4 / 29.3 | 3.46 / 3.47 | 0/0 ; 0/0 |
| 8,192 | FuZev v1.0 (FP32) | 30.1 / 29.9 | 4.53 / 4.53 | 0/0 ; 0/0 |
| 8,192 | FuZev v1.1 (FP32) | 30.5 / 29.8 | 4.51 / 4.52 | 0/0 ; 0/0 |
| 8,192 | ExLlamaV3 1.6.2 (FP32) | 29.3 / 29.1 | 6.21 / 6.13 | 0/0 ; 0/0 |
| 8,192 | ExLlamaV3 1.6.2 (default FP16) | 30.0 / 29.7 | 5.87 / 5.94 | 0/0 ; 0/0 |
| 8,192 | ExLlamaV3 master (FP32) | 29.3 / 29.5 | 5.83 / 6.02 | 0/0 ; 0/0 |
| 8,192 | ExLlamaV3 master (default FP16) | 29.4 / 29.2 | 5.76 / 5.92 | 0/0 ; 0/0 |
| 32,768 | FuZev v1.0 (FP32) | 29.8 / 30.1 | 14.60 / 14.70 | 0/0 ; 0/0 |
| 32,768 | FuZev v1.1 (FP32) | 29.7 / 30.0 | 14.61 / 14.63 | 0/0 ; 0/0 |
| 32,768 | ExLlamaV3 1.6.2 (FP32) | 28.4 / 28.5 | 20.98 / 21.34 | 0/0 ; 0/0 |
| 32,768 | ExLlamaV3 1.6.2 (default FP16) | 29.7 / 30.2 | 20.80 / 20.75 | 0/0 ; 0/0 |
| 32,768 | ExLlamaV3 master (FP32) | 28.7 / 29.7 | 21.02 / 21.05 | 0/0 ; 0/0 |
| 32,768 | ExLlamaV3 master (default FP16) | 29.3 / 29.7 | 20.80 / 20.88 | 0/0 ; 0/0 |

Measured on one NVIDIA GB10 (128 GB unified memory); Python 3.12.3,
PyTorch 2.13.0+cu130, Triton 3.7.1, FlashInfer 0.6.18.post1 (JIT cache
0.6.18.post1+cu130); engines exllamav3-fuzev 1.6.1+fuzev.v1.1,
exllamav3-fuzev 1.5.2+fuzev.v1.0, ExLlamaV3 1.6.2 and ExLlamaV3 master
(058b8630); batch 1, Q8 KV cache, 32,768-token chunks, 98,304-token
context and cache, FP32 recurrent state unless the row says default FP16.
MTP component loaded; prefill uses MTP off, decode uses MTP 3 or off.

The largest between-repetition spread is 16.2 tok/s with MTP 3, tracking
accepted drafts, and 1.03 tok/s with MTP off. These are descriptive ranges
from two repetitions. Source:
[six-build speed records](https://github.com/PeisongXiao/fuzev-research-archive/tree/v1.1/docs/results/comparison/v1.1).

## Workload And Settings

The six configurations below form the FuZev 1.0 head's research study, run on
pre-release builds of the archived research evaluation servers
([archive copy](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/benchmarks-v1.0.md)).
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
  In this study, only this configuration has a TEST read.
- TEST scoring runs offline from extracted features: eight candidate slots,
  8,192-token chunks, FP16 KV. DEV runs call the HTTP server.

| System | Question Accuracy (%) | `breadth-v1` TEST Index | 95% CI |
| --- | ---: | ---: | ---: |
| FuZev 1.0 head | 73.2 | 46.8 | [44.2, 50.0] |
| Frozen-LM label control | 67.0 | 42.4 | [40.2, 45.7] |
| Feature-only head | 72.3 | 45.3 | [42.5, 48.5] |
| [Kev-0.8B (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-0.8b.md#evaluation) | 58.6 | 23.3 | [21.2, 25.9] |
| [Kev-4B (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-4b.md#evaluation) | 69.0 | 38.0 | [35.5, 41.3] |
| [Kev-9B v2 (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-9b.md#evaluation) | 69.8 | 41.0 | [38.8, 43.9] |
| [Kev-27B (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/docs/model-cards/kev-27b.md#evaluation) | 75.7 | 52.3 | [49.2, 55.4] |
| [Jev, published by Kev (Kev 1.0)](https://github.com/jaredpalmer/kev/blob/kev-1.0/README.md#models) | 75.7 | 54.0 | [51.2, 57.0] |

Kev and Jev: results published by the repo owner.

- Paired head minus frozen LM: +4.4 [2.0, 6.4] index points.
- Paired head minus feature-only: +1.5 [-0.5, +3.7].
- Feature-only scores higher on BRIGHT, cfcolor, BFCL and SATA-Bench.

Dataset scores are percentages under DEV's official metric conventions. The
Kev-9B v2 column is `systems["Kev-9B"].datasets` from
[Kev's TEST report](https://github.com/jaredpalmer/kev/blob/kev-1.0/runs/fam-breadth-test-report/report.json).

Kev: results published by the repo owner.

| Dataset | Units | FuZev 1.0 Head | Frozen LM | Feature Only | Kev-9B v2 |
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

- The FuZev 1.0 head trails Kev-9B v2 on ContractNLI, BFCL and API-Bank, and
  ties on SGD / SGD-X.
- The FuZev 1.0 head trails its frozen-LM control on ContractNLI and SGD / SGD-X.

In this offline TEST comparison, FuZev is above
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

In this VL-RewardBench conversion the preferred response is first in 1,226
of 1,247 items (98.3%), so accuracy mostly reflects how often the head
answers Yes or picks the first response.

### Conditional Weighted Results

Each request's weight is the inverse of its subset-inclusion probability times
its source sampling weight. Values are point estimates, without intervals, for
API-eligible represented strata
([evaluation method](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/research-evaluation.md)).

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
- Both VL-RewardBench methods use the same 100 examples. The preferred
  response is first in 98.3% of VL-RewardBench items in this conversion, so
  accuracy mostly reflects how often the head answers Yes or picks the first
  response.

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
the FuZev 1.0 head on features from the EXL3 4.05-bpw backbone.

| Method | Records | Grouped Units | Head (%) | Frozen LM (%) | Paired Difference (pp) [95% CI] |
| --- | ---: | ---: | ---: | ---: | ---: |
| Choice preference | 1247 | 1194 | 68.97 | 89.01 | -20.05 [-22.49, -17.54] |
| Noul relative preference | 1247 | 1194 | 69.85 | 82.60 | -12.75 [-15.15, -10.38] |

- For both methods, head selection accuracy is the same raw, calibrated, and
  when counting equally correct options as hits.
- Intervals use 2,000 paired group-bootstrap draws, seed 17.
- The 100-example evaluation-mix subset above samples this held-out set.
- In this conversion the preferred response is first in 1,226 of 1,247 items
  (98.3%), so accuracy mostly reflects how often the head answers Yes or picks
  the first response.

## Repeat Probes

In the FuZev 1.0 head study, each probe repeats a synthetic two-option decision
immediately and after a different request. Delta columns compare against the
first response. Spread is the largest difference between any two of the three
responses.

| Configuration | Context Tokens | Immediate Repeat | After Other Request | Spread |
| --- | --- | ---: | ---: | ---: |
| ExLlamaV3 EXL3 3.05 / 4.05 / 5.05 bpw | 2,048 / 8,192 / 16,384 / 32,768 | 0 (identical) | 0 (identical) | 0 |
| vLLM NVIDIA NVFP4 | 2,048 | 0.028949 | 0.001992 | 0.0309 |
| vLLM RadixArk NVFP4 | 2,048 | 0.061121 | 0.190495 | 0.2516 |
| vLLM EXL3 4.05 bpw | 2,048 | 0.006018 | 0.011897 | 0.0179 |
| vLLM NVIDIA NVFP4, later trace | 2,048 | 0.104289 | 0.009708 | 0.1043 |

The [later NVIDIA trace](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/repeat/nvidia-later-trace.json)
uses the NVIDIA NVFP4 benchmark run's vLLM engine build with an instrumented
evaluation server. It records a maximum change of 0.10429 from the first response
and 46-48% relative L2 drift in the query features.

Each ExLlamaV3 repeat runs within one server. ExLlamaV3 probabilities can differ:

- between server starts;
- with which prefix pages are already cached;
- with how many options, including those of concurrent requests, share a
  batched forward.

With head 1.1.0, the
[paired bitrate API checks](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/v1.1-head-results.md#paired-bitrate-api)
return identical results for 12 pairs of new and retained-prefix requests per
bitrate. Across 4.05 and 3.05 bpw, 10 of 12 winners and 9 of 12 top-two sets
agree; raw probabilities shift by up to 0.37. In a
[same-prefix probe](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/integration/v1.1/decision-root-concurrency.json),
four three-option questions decided concurrently differ from the same
questions decided one at a time by up to 0.048 raw and 0.042 calibrated
probability; repeated runs of each kind match exactly, and both kinds select
the same option for every question. These checks describe their request set
and settings.

### Run-To-Run Variation

In the FuZev 1.0 head study, three runs of the vLLM NVIDIA NVFP4 configuration
share 670 identical evaluation-mix requests. Two runs are partial; their
completed responses supply this comparison ([archive records](https://github.com/PeisongXiao/fuzev-research-archive/blob/v1.1/docs/results/README.md#v10-six-quality-panels)).

- Between any two runs, 14.6-14.9% of selections differ and probabilities differ
  by up to 0.62.
- Per-run accuracy on those requests is 73.4%, 74.8% and 75.2%.

No other configuration has multiple runs with completed decisions. Sampling
intervals and repeat probes describe different sources of uncertainty.

## Limitations

- Study results describe pre-release builds of the archived research evaluation
  servers with their recorded settings.
- Each served quality configuration has one pass; head 1.1.0 seed variance is
  unmeasured.
- The study's three vLLM quality runs predate two vLLM-fork changes that affect decision
  features: CUDA-graph replay of per-layer embeddings, and KV-cache slot mapping.
  These runs were not repeated on the published build.
- The study's ExLlamaV3 quality runs use a pre-release fork build. Its extension predates
  the released CUDA-graph kernel-argument fix, and its optimized prefill casts
  gated-delta-network operands to FP16. These runs were not repeated on the released build.
- vLLM repeat variation can exceed differences between configuration indices.
- Image decisions are experimental. VL-RewardBench answer order limits what
  its accuracy says about response preference; second-preferred counts are
  reported with the converted-request scores.
- Head 1.1.0 validates image Choice at 2 options. Image Noul, image Score,
  larger image option sets and text Score quality are unvalidated.
- Speed results describe the recorded builds and settings. Served API overhead,
  concurrency and longest-context performance are unmeasured.

See also:

- [API limitations](api.md#limitations): option counts and supported engine settings.
- [Performance](performance.md): prefill timings with all repetitions.
- [Integration](integration.md): generation and MTP checks.
