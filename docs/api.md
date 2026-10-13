# Decision API

`POST /v1/systemone` answers Choice, Boolean (Noul) and ordered Score questions
in the public Jev `state`, `questions` and `answers` format. The
[vLLM](https://github.com/PeisongXiao/fuzev-vllm-plugin) and
[ExLlamaV3](https://github.com/PeisongXiao/fuzev-exllamav3-plugin) integrations
serve decisions and generation from the same running model.

## Usage

Connect to a server with a decision head loaded; the default address is
`http://127.0.0.1:8000`. Install the public Python client:

```bash
python3 -m venv .venv-client
. .venv-client/bin/activate
python -m pip install typesafe-sdk
```

```python
from typesafe_sdk import Choice, TypeSafeClient

with TypeSafeClient(
    api_key="local", base_url="http://127.0.0.1:8000", model="jev"
) as client:
    result = client.system_one(
        state={"message": "I was charged twice."},
        questions={
            "team": Choice(
                instructions="Which team should handle this ticket?",
                criteria={"billing": "Charges", "shipping": "Delivery"},
            ),
        },
    )
    print(result.choices["team"].choice)
    print(result.request_id)
```

- **Timeout.** The SDK times out after 10 seconds and retries twice. For long
  states or many options, pass a larger `timeout` (seconds) to `TypeSafeClient`.
- **Authentication.** The default local server runs without authentication;
  `local` fills the SDK's required key. With `--api-key`, pass that key to the
  SDK and send `Authorization: Bearer <key>` on HTTP requests.

The same request with curl:

```sh
curl --fail-with-body --silent --show-error \
  http://127.0.0.1:8000/v1/systemone \
  -H 'Content-Type: application/json' \
  --data '{
    "model": "jev",
    "state": {"message": "I was charged twice."},
    "questions": {
      "team": {
        "type": "choice",
        "instructions": "Which team should handle this ticket?",
        "criteria": {"billing": "Charges", "shipping": "Delivery"}
      }
    }
  }'
```

Illustrative response, with example probabilities and token counts:

```json
{
  "model": "my-model",
  "answers": {
    "team": {
      "type": "choice",
      "choice": "billing",
      "confidence": 0.8,
      "probabilities": {
        "billing": 0.9,
        "shipping": 0.1
      }
    }
  },
  "usage": {
    "input_tokens": 120,
    "output_tokens": 40
  },
  "latency_ms": 12.5
}
```

| Field | Meaning |
| --- | --- |
| `model` | Serving model name (`--model-name`); `my-model` matches the [server setup](install.md#start-a-server) |
| `answers` | Answers keyed by submitted question key |
| `usage.input_tokens` | Logically submitted decision tokens, including prefixes repeated across option branches |
| `usage.output_tokens` | Serialized answer length in model tokens |
| `latency_ms` | Request duration in milliseconds (FuZev extension) |

Responses carry `X-TypeSafe-Request-ID`; the SDK exposes it as
`result.request_id`.

## Answers

| Question | Criteria | Answer |
| --- | --- | --- |
| Choice | Object mapping option names to descriptions | `choice`: selected name; `probabilities`: distribution by name; `confidence`: concentration above the uniform baseline |
| Noul | Optional `true` and `false` descriptions | `noul`: probability of Yes/true |
| Score | Ordered array of level descriptions | `score`: expected zero-based level; `legend`: supplied levels; `probabilities`: distribution by index; `confidence`: concentration around the most probable level |

- Every answer includes its `type`.
- In the SDK, build questions with `Choice`, `Noul` and `Score` from
  `typesafe_sdk`. Read answers from `result.choices[name]`,
  `result.nouls[name]` and `result.scores[name]`.
- Score legend values preserve supplied strings, objects and arrays. Legend and
  probability keys are strings in JSON and integers in the SDK.

State and option text use Kev-style rendering, preserving order:

- Strings stay literal.
- Objects become indented `key: value` text.
- Arrays become `- item` lines.
- Omitted instructions become empty text.
- Choice descriptions follow their option names.
- Noul options are `[Yes, No]`, in that order. Any `true` and `false`
  descriptions go in the matching option text.

Each question is scored from the shared state alone. The response preserves
question order.

Scoring uses the server's calibration mode (see [model configuration](models.md)):

| Mode | Applies | Used by |
| --- | --- | --- |
| `raw` | Temperature 1 | FuZev servers (default) and the published benchmarks |
| `calibrated` | The head bundle's temperatures | Kev by default; FuZev servers started with `--calibration calibrated` |

Confidence formulas:

- **Choice**, with `n` options: `clamp((p_selected - 1/n) / (1 - 1/n), 0, 1)`.
- **Score**: `max(0, 1 - sum_i(p_i * abs(i - selected)) / D)`, where
  `D = sum_i(abs(i - (n-1)/2)) / n` and `selected` is the most probable
  zero-based level.

## Errors

Both integrations use these statuses:

| Status | Meaning |
| --- | --- |
| 401 | Missing or incorrect bearer token, when authentication is configured |
| 404 | Missing, released or evicted prefix; unknown cancellation ID; or a route unavailable without a loaded head |
| 409 | The supplied request ID is already active |
| 413 | Request body exceeds the configured byte limit |
| 422 | Invalid JSON; invalid value for a known request field; unknown `/v1/decisions` field; invalid request ID; option count outside the head's supported range; input exceeding the model's context window |
| 499 | Request cancelled or client disconnected |
| 500 | Unexpected server error |
| 503 | Server shutting down or engine unavailable |

Validation errors return a `detail` list. For a 17-option Choice on the
published head, it names the affected field:

```json
{
  "detail": [
    {
      "loc": ["body", "questions", "team", "choice", "criteria"],
      "msg": "This head supports at most 16 options",
      "type": "value_error"
    }
  ]
}
```

A missing or evicted prefix instead returns 404 with an OpenAI-style
`{"error": {"type": "invalid_request_error", "message": "Unknown or evicted prefix"}}`
envelope.

Independent questions queue through the engine's scheduler. If a question fails
or the request is cancelled, the server cancels the request's remaining work.

## Extensions

FuZev adds these behaviors beyond the public Jev contract.

| Extension | Type And Location | Behavior |
| --- | --- | --- |
| `latency_ms` | Number in every SystemOne response | Elapsed request time in milliseconds |
| `X-TypeSafe-Request-ID` | Optional request header; echoed response header | Set it before submitting to enable cancellation. Characters: `[A-Za-z0-9._:-]+`. An active duplicate returns 409. If omitted, the server assigns one. |
| `POST /v1/requests/{id}/cancel` | Optional; takes a request ID | Returns 202 with `{"id": "...", "cancel_requested": true}`. Cancels all pending work and waits for it to drain; the original request then ends with 499. A started stream ends with an error event and `data: [DONE]`. Repeats return 202 during cleanup, then 404. |
| `DELETE /v1/prefixes/{id}` | Optional; takes a prefix ID | Releases a retained prefix; returns `{"id": "...", "released": true}` |
| `retain_decision_prefix` | Optional boolean on generation requests, default `false` | When true, the generation response includes a string `prefix_id` for its accepted prompt and completion tokens |
| `prefix_id` | Optional string on SystemOne and decisions requests | Uses retained generation tokens as context: SystemOne appends the rendered `state`; `/v1/decisions` appends `additional_context`. Release the prefix when done. |
| `image` | Optional string on SystemOne, decisions and generation requests | Experimental: a `data:image/<type>;base64,...` URL or bare base64. Chat completions also accept one OpenAI `image_url` content part in either format. |
| `/v1/decisions` | Optional lower-level endpoint | Takes explicit candidate text; returns per-candidate scores ([fields](#lower-level-decisions)) |
| `model` on SystemOne | Optional non-empty string | Accepted and ignored; responses name the configured serving model |
| Unknown SystemOne fields | Extra top-level or per-question fields | Accepted and ignored; known fields keep type and value validation |
| Score levels 11-16 | Extra levels in the criteria array | The published head supports them; hosted Jev documents up to 10 |

### Retained Prefixes

- Both engines retain up to 128 prefixes by default; set the count with
  `--prefix-capacity`.
- Entries have no time-based expiry. Retaining past capacity evicts the least
  recently used entry; each access refreshes an entry's LRU position.
- A missing, released or evicted ID returns 404 `invalid_request_error`.
- Prefixes from generation requests that included an image keep the image
  embeddings with the accepted tokens.

### Images

Send a local `image.png` through the SDK's `extra_body`:

```python
import base64
from pathlib import Path

from typesafe_sdk import Noul, TypeSafeClient

image = base64.b64encode(Path("image.png").read_bytes()).decode("ascii")
with TypeSafeClient(
    api_key="local", base_url="http://127.0.0.1:8000", model="jev"
) as client:
    result = client.system_one(
        state="Inspect this image.",
        questions={"text": Noul(instructions="Does the image contain text?")},
        extra_body={"image": f"data:image/png;base64,{image}"},
    )
    print(result.nouls["text"].noul)
```

- The server decodes the image in memory and passes RGB pixels to the model's
  image processor.
- A multi-frame image uses its first frame.
- Images above Pillow's decompression-bomb limit (`PIL.Image.MAX_IMAGE_PIXELS`,
  about 89 million pixels by default) return 422.

### Lower-Level Decisions

`POST /v1/decisions` accepts explicitly rendered inputs.

| Request Field | Meaning |
| --- | --- |
| `model` | Optional non-empty alias; responses report the configured serving model |
| `primitive` | `choice`, `noul` or `ordinal` (ordered Score) |
| `question` | Instruction text; may be empty |
| `candidates` | Ordered array of unique `id` and `text` pairs; array order defines the ordinal levels |
| `messages` or `prefix_id` | Supply standalone messages or a retained prefix |
| `additional_context` | Optional text appended after the retained prefix |
| `head_id` | Optional string, accepted and ignored; responses report the loaded head's `head_id` |
| `image` | Optional experimental image input, as in [Images](#images) |

Noul candidates are `[yes, no]`, in that order, with text `Yes` and `No`, each
optionally followed by its criteria description.

| Response Field | Meaning |
| --- | --- |
| `id` | Request ID |
| `model` | Configured serving model name |
| `primitive` | Requested decision primitive |
| `head_id` | Loaded decision head identifier |
| `calibration_id` | `raw` or `calibrated` |
| `selected_id` | Candidate ID with the highest probability |
| `scores` | Ordered records of `id`, `logit`, `raw_probability` and `calibrated_probability`; in raw mode both probability fields hold the raw probability |
| `execution.submitted_tokens` | Logically submitted decision tokens |

### Generation And Discovery

`POST /v1/chat/completions` and `POST /v1/completions` return OpenAI-style
responses. Chat answers appear in `message.content` and model thinking in
`message.reasoning`:

- vLLM splits them with the reasoning parser named by `--reasoning-parser`
  (default `qwen3`; see the
  [plugin configuration](https://github.com/PeisongXiao/fuzev-vllm-plugin/blob/main/README.md#configuration)).
- ExLlamaV3 splits at Qwen-style `<think>` and `</think>` tags.

Set `chat_template_kwargs.enable_thinking` to control thinking in the model's
chat template. Example chat request body:

```json
{
  "model": "my-model",
  "messages": [{"role": "user", "content": "Explain a duplicate charge briefly."}],
  "chat_template_kwargs": {"enable_thinking": false},
  "stream": true
}
```

Stream a chat response:

```bash
curl --fail-with-body -N http://127.0.0.1:8000/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"my-model","messages":[{"role":"user","content":"Say hello."}],"max_tokens":32,"stream":true}'
```

With `stream: true`, both endpoints return `text/event-stream`:

| Event | Content |
| --- | --- |
| Chat chunk | `object: "chat.completion.chunk"`, with `choices[0].delta.content` and optional `delta.reasoning` |
| Completion chunk | `object: "text_completion"`, with `choices[0].text` |
| Final chunk | `finish_reason` and usage, followed by `data: [DONE]` |
| Started stream cancelled | Last event before `data: [DONE]` is `{"error": {"message": "...", "type": "request_cancelled", "code": 499}}` |
| Started stream failed | The same error event, with type `server_error` and the HTTP status code |

Disconnecting cancels the engine request.

| Request Field | Behavior |
| --- | --- |
| `model` | Optional non-empty alias; responses report the configured serving model |
| `messages` / `prompt` | Supply one: chat messages or a text-completion prompt |
| `messages[].role`, `content` | Content: a string, or an array of `text` parts (joined) and at most one `image_url` part. Role `developer` becomes `system` |
| `max_tokens` / `max_completion_tokens` | Optional positive output budget; `max_completion_tokens` takes precedence. Default: the remaining context; vLLM first applies a model default budget, capped at the remaining context |
| `temperature`, `top_p`, `seed` | Optional sampling (`temperature` 0 to 2, `top_p` above 0 up to 1). Omitted values use the model's `generation_config.json`, else engine defaults |
| `stop` | String or array of up to four stop strings; null means none |
| `stream` | Boolean, default false; true selects SSE |
| `top_k` | FuZev extension: optional top-k sampling; 0 disables it |
| `chat_template_kwargs` | FuZev extension: overrides startup template settings for messages; ignored for raw `prompt` requests |
| `retain_decision_prefix` | FuZev extension: retain accepted prompt and completion tokens; default false |
| `image` | FuZev extension; formats as in [Extensions](#extensions). Chat requests may instead send one OpenAI `image_url` content part; a request with two images returns 422. Placement: start of the first user message, or a new user message after the existing ones when there is none |
| `n` | One completion per request (`n=1`) |

- Unknown request and message fields are accepted and ignored; known fields are
  validated.
- Requests for tool calls, structured `response_format`, log probabilities or
  multiple completions return 422.

| Response Field | Behavior |
| --- | --- |
| `id`, `object`, `created`, `model` | Request ID, response kind, Unix timestamp and configured model |
| `choices[].index`, `finish_reason` | Completion index and stop/length reason |
| `choices[].message.content` / `choices[].text` | Chat answer or text completion |
| `choices[].message.reasoning` | FuZev extension: model thinking, separated from answer text |
| `usage.prompt_tokens`, `completion_tokens`, `total_tokens` | Token accounting |
| `usage.prompt_tokens_details.cached_tokens` | vLLM-only extension: engine-reported cached input tokens |
| `prefix_id` | FuZev extension: retained prefix ID; in streaming, supplied on the final chunk |

Use the model name from `GET /v1/models`. Both integrations return
`{"object": "list", "data": [...], "models": [...]}`:

- `data` is the OpenAI model list.
- `models` holds SDK records with `name`, `description` and `release_date`; the
  release date comes from the head bundle. With no head loaded, `models` is
  empty and the SystemOne route is unavailable.

`GET /health` returns `status`, `model`, `context_window` and `capabilities`
(keys `generation`, `decisions`, `images` and `streaming`) on both engines. The
health check is unauthenticated; `--api-key` protects `/v1/` routes.

## Capacity Comparison

| API | Choice Options | Score Levels | Noul Outcomes |
| --- | --- | --- | --- |
| FuZev | 2 to the head's `max_options` (16 for the published head) | 2 to the head's `max_options` (16 for the published head) | 2 |
| [Hosted Jev](https://docs.typesafe.ai/api) | Up to 255 | Up to 10 | 2 |
| [Kev](https://github.com/jaredpalmer/kev#api) | 1-255 | 1-255 descriptions | 2 |

## Limitations

FuZev v1.0 is an experimental research release.

- **Option counts.** v1.0 accepts 2 to 16 Choice options or Score levels per
  question. The head bundle declares the maximum (`max_options` in its
  `config.json`): 16 for the published head. Larger counts are planned for
  v1.1. Noul has exactly two outcomes. Hosted Jev and Kev allow different
  counts; see the [capacity comparison](api.md#capacity-comparison).
- **Tested platform.** NVIDIA GB10 (Python 3.12, PyTorch 2.13.0+cu130). Both
  engines need CUDA; the head runs on CPU or GPU (`--head-device`).
- **vLLM.** Readout supports the V1 GPU model runner
  (`VLLM_USE_V2_MODEL_RUNNER=0`), synchronous scheduling
  (`"async_scheduling": false` in `engine.json`) and pipeline-parallel size 1.
  Configuration accepts KV and encoder-cache transfer connectors; live readout
  with them is unverified. MTP with EXL3 checkpoints is unsupported in v1.0.
  See the
  [engine limitations](https://github.com/PeisongXiao/vllm-fuzev/blob/main/docs/design/fuzev.md#limitations).
- **ExLlamaV3 readout.** Supports Generators without tensor parallelism.
  Multi-GPU layer splits are untested.
- **Speculative decoding.** With MTP or another speculative method that keeps a
  draft KV cache, vLLM decisions recompute the full context per option: about
  `K+1` full context prefills for `K` options. Serve decision-heavy workloads
  without speculative decoding.
- **Decision cost.** A `K`-option decision runs one pass over the context plus
  one pass per option. Each option pass reuses cached context up to the
  engine's cache block: 1,568 tokens on vLLM with BF16 KV for Qwen3.8, and 256
  tokens on ExLlamaV3. For contexts shorter than one block, a decision costs
  about `K+1` times a plain prefill of that context.
- **Repeatability.** Decision probabilities vary between repeated requests on
  vLLM. On ExLlamaV3 they can differ slightly between a request's first run and
  later repeats (once its prefix pages are cached) and between server starts.
  The cause: Triton selects kernel configurations by timing when it first
  compiles them. See the [repeat probes](benchmarks.md#repeat-probes).
- **Listener and request size.** The listener defaults to `127.0.0.1:8000` with
  no key; change it with `--host`, `--port` and `--api-key`. Request bodies
  default to 8 MiB (`--max-request-bytes`); base64 images count toward this
  limit. Generation accepts up to four stop sequences.
- **Images** are experimental.
  - Image decisions use the head's Qwen-format encoding; generation requests
    with an image use the served model's own image placeholder.
  - ExLlamaV3 needs `--vision` or the `image_embeddings` callback of
    `build_generator_service`.
  - Weak visual tasks include MM-RLHF appropriateness and relative preference, and
    VL-RewardBench Choice and relative-preference Noul; see
    [benchmarks](benchmarks.md).
  - The published builds' image path has CPU test coverage only. The image
    results in [benchmarks](benchmarks.md) come from the research evaluation
    servers.

Research methods and measured scope are in the
[archive](https://github.com/PeisongXiao/fuzev-research-archive).
