# Inference Proxy

The inference proxy handles all LLM inference for agents running in the sandbox. Agent inference can run through Chutes or OpenRouter with OpenAI API compatibility.

Proxy summaries record aggregate inference activity for a validator job run. Every tracked proxy request is assigned to the job run identified by its request headers, then grouped by execution or evaluation phase, requested model, provider-returned model, and proxy provider. The summary contains request outcomes, outbound LLM attempts, retries, tokens, timing, and status codes; it does not contain prompts or responses.

## Capabilities

The proxy supports:

- **Tool use** - Agents can make function calls through the proxy
- **Multi-turn** - Agents can have multi-turn conversations
- **Reasoning** - Reasoning model replies are supported

All OpenAI API compatible calls are available. Ask us for agent coordination libraries and we'll add them.

## Access

Agents run in a sandboxed environment. Internet access is restricted — all external calls go through the inference proxy.

You will need an `INFERENCE_API_KEY` to use the proxy for agent execution. This can be a Chutes key or an OpenRouter key, and the provider is detected automatically.

For miners, that key is used for local development and can also be attached when submitting an agent. The miner-submitted key is encrypted at rest in the platform, and validators receive it to run the agent, so the miner pays for inference. Validators still use their own `CHUTES_API_KEY` for evaluation/scoring.

## Adding tracking headers to an agent

Custom agents that call the inference proxy directly should pass the job-run and phase headers with every request. The validator injects the current job run into the agent container as the `JOB_RUN_ID` environment variable. The baseline `miner/agent.py` reads that value and sends it like this:

```python
headers = {
    "x-inference-api-key": self.inference_api_key,
    "x-agent-id": self.agent_id,
    "x-job-run-id": self.job_run_id,
    "x-request-phase": "execution",
}
```

The tracking headers are:

- `x-job-run-id`: associates the request with the current validator job run. Read it from `JOB_RUN_ID` rather than hard-coding it. A missing or blank value disables summary tracking for the request.
- `x-request-phase`: groups the request under `execution`, `evaluation`, or `unknown`. Miner-agent requests should use `execution`; the validator scorer uses `evaluation`.

`x-agent-id` supplies logging context.

## Downloading proxy summaries

The agent detail page has a **Download proxy summaries** button in the **Agent Runs** section. It downloads:

```text
agent-<agent_id>-job-run-summaries.json
```

The export contains aggregate proxy telemetry for the agent. It is grouped first by validator, then by job run. A job run can contain several summary rows because rows are separated by phase, requested model, model reported by the provider, and proxy provider.

```json
[
  {
    "validator_id": 30,
    "job_runs": [
      {
        "job_run_id": 20,
        "summaries": [
          {
            "id": 10,
            "job_run_id": 20,
            "agent_id": 1,
            "phase": "execution",
            "req_model": "qwen/qwen3.6-35b-a3b",
            "model": "qwen/qwen3.6-35b-a3b-20260415",
            "provider": "openrouter",
            "requests": 8,
            "success": 7,
            "error": 1,
            "llm_requests": 10,
            "llm_success": 7,
            "llm_error": 3,
            "retries": 2,
            "input_tokens": 800,
            "output_tokens": 400,
            "cached_tokens": 120,
            "duration_ms_total": 24000,
            "duration_ms_avg": 3000,
            "duration_ms_max": 8000,
            "status_codes": {
              "200": 8,
              "429": 2
            },
            "generated_at": "2026-07-23T12:00:00",
            "created_at": "2026-07-23T12:00:01",
            "updated_at": "2026-07-23T12:00:01"
          }
        ]
      }
    ]
  }
]
```

An agent without recorded proxy summaries produces an empty array.

### Export structure

| Field | Meaning |
| --- | --- |
| `validator_id` | Platform ID of the validator that ran the grouped job runs. |
| `job_runs` | Job runs for this agent and validator that have proxy telemetry. |
| `job_run_id` | Platform ID of the job run. It appears both in the job-run wrapper and in each summary row. |
| `summaries` | Aggregate rows for the job run, ordered with execution before evaluation. |

### Summary fields

| Field | Meaning |
| --- | --- |
| `id` | Platform database ID of the aggregate summary row. |
| `job_run_id` | Job run to which the telemetry belongs. |
| `agent_id` | Agent to which the telemetry belongs. |
| `phase` | `execution` for the miner agent, `evaluation` for validator scoring, or `unknown` when the request did not contain a recognized phase. |
| `req_model` | Model name requested by the agent or evaluator. |
| `model` | Model name returned in the provider response. See [Requested and returned models](#requested-and-returned-models). |
| `provider` | Proxy provider selected from the inference API key, currently `openrouter` or `chutes`. This is not necessarily the underlying provider to which an aggregator routed the request. |
| `requests` | Number of completed requests received by the Bitsec inference proxy. |
| `success` | Proxy requests that ultimately returned successfully to the caller. |
| `error` | Proxy requests that ultimately failed, after any retries. |
| `llm_requests` | Outbound LLM attempts, including the first attempt and any retries. |
| `llm_success` | Outbound attempts with an HTTP status from 200 through 399 and no response-level error detected by the proxy. |
| `llm_error` | Outbound attempts that failed at the transport, HTTP, response-format, or usable-content level. See [What counts as an LLM error](#what-counts-as-an-llm-error). |
| `retries` | Number of outbound retry attempts. The original attempt is not a retry. |
| `input_tokens` | Sum of provider-reported prompt tokens across recorded outbound attempts. |
| `output_tokens` | Sum of provider-reported completion tokens across recorded outbound attempts. |
| `cached_tokens` | Sum of provider-reported cached prompt tokens across recorded outbound attempts. |
| `duration_ms_total` | Total end-to-end proxy request time in milliseconds, including proxy queueing, retries, and retry backoff. |
| `duration_ms_avg` | `duration_ms_total / requests`, stored as a whole number of milliseconds. |
| `duration_ms_max` | Longest end-to-end proxy request time in the row, in milliseconds. |
| `status_codes` | Counts by status code. The flat format contains outbound LLM codes; newer validator telemetry can separate proxy and LLM codes. See [Status-code formats](#status-code-formats). |
| `generated_at` | Time at which the validator generated the summary snapshot. |
| `created_at` | Time at which the platform first stored the row. |
| `updated_at` | Time at which the platform last updated the row. |

For each row, `requests = success + error` and `llm_requests = llm_success + llm_error`. `llm_requests` can be greater than `requests` because one proxy request can make several outbound attempts. Similarly, `retries` counts attempts, not requests that needed a retry.

### What counts as an LLM error

`llm_error` is broader than an HTTP error count. It includes:

- connection failures where no HTTP response was received;
- non-successful provider HTTP responses, including rate limits;
- invalid JSON or an unexpected response structure, such as a missing `choices` value;
- a response rejected by the proxy as unusable because its finish reason is `length` or `content_filter`.

The last two cases can have an LLM status code of `200`. For example, a provider can return HTTP `200` with `finish_reason: "length"` after reaching the output-token limit. The proxy records that outbound attempt as an `llm_error` and returns a proxy error to the caller. Consequently, the number of LLM `200` responses can be greater than `llm_success`.

Token totals include any usage reported with these unusable responses. They can also include more than one attempt for a request. Failed connections and provider errors often report no usage, so a zero token count does not show that no attempt was made.

### Requested and returned models

`req_model` preserves what the caller requested. `model` uses the exact model identifier returned by the provider when one is available. Providers can resolve an alias to a dated version, route to another compatible model identifier, or otherwise return a value that differs from the request. This can produce more than one summary row for the same `req_model`.

If an attempt fails before a provider response supplies a model—for example, a connection error, timeout, or provider HTTP error—`model` falls back to `req_model`. The error is therefore grouped under the requested model, even though the request may never have reached the underlying model.

When retries eventually produce a response containing a model, the attempts buffered for that proxy request are grouped under the final returned model. The export does not expose the underlying routed provider for each individual attempt.

### Status-code formats

Newer sandbox telemetry distinguishes the two HTTP boundaries:

```json
{
  "status_codes": {
    "proxy": {"200": 7, "502": 1},
    "llm": {"200": 8, "429": 2}
  }
}
```

- `proxy` contains the statuses returned by the Bitsec proxy to the agent or evaluator.
- `llm` contains statuses from outbound provider attempts. A synthetic code of `0` means that no HTTP response was received.

The platform download can contain a flat map:

```json
{
  "status_codes": {"200": 8, "429": 2}
}
```

In the flat format, the values are outbound LLM status codes; proxy response codes were not recorded separately. The export returns stored telemetry, so the exact shape depends on the validator and platform versions that recorded and stored each row.

### Telemetry caveats

- These are aggregates, not full request or response logs. They cannot show which prompt, project, or individual attempt caused a count.
- Metrics are recorded only when the proxy request has a job-run ID. Requests without one are not included.
- Collection and submission are best effort. A validator can complete a job run even if proxy metrics are unavailable or cannot be submitted, so a missing row does not prove that no inference occurred.
- Empty or invalid metadata can be reported as `unknown`.
- Provider usage fields are not uniform. Missing token or cache data is recorded as zero.
- The average duration is integer-valued and may discard a fractional millisecond.
- The metrics are operational telemetry and should not be treated as billing records or an authorization boundary.
