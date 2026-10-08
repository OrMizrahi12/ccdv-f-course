# Module 1 on one page: Model & Technical Foundations

CCDV-F · the exam's Model Selection and Optimization domain (16.8%, about 9 of 53 items) · checked against the Claude docs on 2026-10-08

| Lesson | Keep this |
|---|---|
| 1.1 Tokens | Claude writes one token at a time. Input, output and thinking share one context window. Count before you send, and recount when the model changes: Claude 4.7 and later count the same text as about 30% more tokens. |
| 1.2 Sampling | The same request can return different text. Test with several runs and a pass rate, never one golden string. On Claude 4.7 and later, a non-default `temperature`, `top_p` or `top_k` is a 400: use prompts, structured outputs and effort. |
| 1.3 The request | `POST /v1/messages` with your key (`x-api-key`, or `Authorization: Bearer`), `anthropic-version`, and a JSON body: `model`, `max_tokens`, `system` (top level), `messages`. A 4xx is your request: fix it. 429 and 5xx pass: retry with backoff (the SDK retries twice by default). |
| 1.4 The response | Content blocks by `type`, `stop_reason`, `usage`, the request ID (`request-id` header). Check `stop_reason` before you trust the text: `max_tokens` is truncated, `tool_use` wants a `tool_result`, `refusal` is an HTTP 200. The API is stateless: send the whole history. |
| 1.5 Streaming and async | SSE events: `message_start` → block start / deltas / stop → `message_delta` (stop reason, cumulative usage) → `message_stop`, plus `ping`. A broken stream: keep the partial text and continue. Many calls: the async client plus a semaphore. |
| 1.6 Choosing a model | Start with the smallest model that passes your evals, and step up on evidence. Pin the ID you tested. Watch `lifecycle` and the retirement date; before switching, follow the migration guide and rerun your evals. |
| 1.7 Reasoning | Model and reasoning are two dials. Today: `thinking: {"type": "adaptive"}` plus `output_config.effort`; `max_tokens` is the hard cap. Thinking tokens are billed as output, shown or omitted. The older `budget_tokens` is a target, rejected on Claude 4.7 and later. |
| 1.8 Money | Price = input × in-price + output × out-price (+ cache writes 1.25×, reads 0.1×). Count first, log every request, compare models on cost per completed task. Free wins: caching, trimming, Batch (50% off). Tradeoffs need evals: model tier, effort, output length. |

## The four traps

| Trap | It sounds like | Why it's wrong |
|---|---|---|
| Retry it | "Add retries with backoff." | Retries fix what passes on its own (429, 5xx, timeouts), never a 400 or a reply cut off by `max_tokens`. |
| Make it bigger | "Use the most capable model." "Raise `max_tokens`." | Bigger costs more and doesn't lift a cap you set, a rate limit you hit, or a retirement date. |
| Yesterday's API | "Set temperature to 0." "Give it a thinking budget." | Right once; today's models reject some of these with a 400. The exam expects today's answer. |
| The wrong yardstick | "Pick the lowest price per token." "Stream it." | A real measure, but not what the question asks: cost per finished task, a pass rate, calls in flight. |

Sources: platform.claude.com/docs (Errors, Context windows, Streaming messages, Choosing the right model, Models API, Model deprecations, the Claude Haiku 5.5 migration guide, Building with thinking, Effort, Pricing, Prompt caching, Token counting, Batch processing, Optimizing for cost and intelligence) and the CCDV-F exam guide. API details change: confirm them in the official docs.
