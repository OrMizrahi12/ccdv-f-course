# Lesson 9.4: Where it runs

Claude Certified Developer – Foundations (CCDV-F) · Module 9: Shipping & Reuse · Lesson 4

This folder holds the files from the lesson, exactly as they were committed on the recording machine on 2026-10-12. It also has the steps to repeat the lesson in your own `quiz-forge` project, starting from the end of Lesson 9.3. The commits:

- `e856c9c` "Where it runs: a platform switch in the client factory, and the gaps per platform"
- `aabdddb` "Latency measured from here, and ADR 0001: where Quiz Forge runs"

By the end of the lesson you can:

- compare the five places Claude runs;
- switch the client for a platform;
- defend the choice to security and procurement.

**Quick check:** before you move on, take this lesson's quick check. It's a few short questions, and it takes a couple of minutes.

## What's in this folder

| File | What it is |
|---|---|
| `quizforge/platforms.toml` | The five platforms as data, each block from its docs page (checked 2026-10-10): the SDK's client class, who operates it, who bills, how residency works, the Haiku 5.5 model ID, and `missing`: the features Quiz Forge uses that the platform's "Features not supported" list names. |
| `quizforge/platforms.py` | Reads the table: where Quiz Forge uses each feature, the current platform (`[platform] name` in `config.toml`), `base_client()` (the SDK class for that platform) and `gaps()`. `python -m quizforge.platforms` prints what would break where. |
| `lesson.diff` | The two small edits to existing files: `quizforge/client.py` builds its base client through `platforms.base_client()`, and `config.toml` gets `[platform] name = "anthropic"`. Apply it with `git apply lesson.diff`, or type the lines yourself. |
| `tests/test_platforms.py` | Tests that need no cloud account: each platform's client class is replaced by a stand-in that records how it was built. |
| `experiments/latency.py` | Ten tiny streaming requests through `make_client` on generate's model: time to the first token and to the end, p50 and p95. Point `[platform]` at another platform to compare. |
| `docs/adr/0001-where-it-runs.md` | The decision record: context, options, decision and consequences. |

## Before you start

- Your `quiz-forge` project at the end of Lesson 9.3, on `main`, with a clean `git status`.
- Your API key in `.env`, for `latency.py` and one `generate`.
- No cloud account is needed: the platform tests use stand-ins. To actually run on another platform, you'd need that platform's account, credentials and model access, set in its own environment variables.
- Cost: ten tiny requests in `latency.py`, plus one `generate` for one lesson (about $0.002 on the recording machine).

## Steps (the lesson's order)

1. **The platforms, as data:** add `quizforge/platforms.toml`. Check each block against the platform's current docs page; feature lists change.
2. **The switch:**
   1. Add `quizforge/platforms.py`.
   2. In `quizforge/client.py`, add `platforms` to the `from quizforge import ...` line, and build the base client with `client = platforms.base_client(asynchronous)`.
   3. At the end of `config.toml`, add `[platform]` with `name = "anthropic"`.
3. **What breaks where, and tests:**
   1. Run `.venv/bin/python -m quizforge.platforms`.
   2. Add `tests/test_platforms.py`, run `.venv/bin/python -m pytest -q`, then commit.
4. **Measure, run, decide:**
   1. Add `experiments/latency.py`, and run `.venv/bin/python -m experiments.latency`.
   2. Run one real request through the factory: `.venv/bin/python -m quizforge generate --lesson L1.3`.
   3. Write `docs/adr/0001-where-it-runs.md` for your own requirements, then commit.

On Windows, the venv's Python is `.venv\Scripts\python`.

## What the lesson printed

These results are from the recording machine (2026-10-12). Latency depends on where you are; your numbers will differ.

**What breaks where:**

```
anthropic  Anthropic                billed by Anthropic          residency: inference_geo per request; workspace defaults
           ok      everything Quiz Forge uses
aws        Anthropic, on AWS        billed by AWS                residency: inference_geo per request
           ok      everything Quiz Forge uses
bedrock    AWS                      billed by AWS                residency: the region of the endpoint
           breaks  structured_outputs  generate, repair, the screen (2.4, 8.1)
           breaks  files               qf evidence (3.4)
           breaks  batches             qf batch (3.5)
           breaks  models              qf model, the pinned-model check (1.6)
google     Google Cloud             billed by Google Cloud       residency: the region of the endpoint
           breaks  files / batches / models
foundry    Microsoft or Anthropic   billed by Azure Marketplace  residency: the US Data Zone deployment type
           breaks  batches / models / files
```

**The tests:** `304 passed, 4 deselected`.

**Latency, from the recording machine:**

```
anthropic / claude-haiku-5-5, 10 requests
first token  p50 1.03 s   p95 1.48 s
whole reply  p50 1.04 s   p95 1.48 s
```

**One real request through the factory:** `L1.3: passed after 1 attempt(s), $0.00159`.
