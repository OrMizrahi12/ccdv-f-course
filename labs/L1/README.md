# Lab 1: Quiz Forge v0, a streaming CLI with a usage and cost log

Claude Certified Developer – Foundations (CCDV-F) · Module 1: Model & Technical Foundations · the module checkpoint

In Module 1 you wrote one experiment per lesson. In this lab you turn them into the first version of Quiz Forge: a command line tool that reads one lesson from the course library, counts its tokens first, refuses anything over a cost limit, streams practice questions to the terminal, recovers if the stream breaks, and logs the tokens, the request ID and the cost of every request. With several lessons, it runs the calls at once, with a limit. The same call is also written in TypeScript.

You pass the lab when the self-check tests pass and a real run writes questions and a usage log.

## What you start from

`starter/` is the `quiz-forge` project exactly as it stands after Lesson 1.8: `experiments/` from Lessons 1.1 to 1.8, the library snapshot in `library/ccao-f/v1/`, `ts/` from Lesson 1.3, the pinned `requirements.txt`. If your own project is there too, keep working in it; if not, copy `starter/` to `~/projects/quiz-forge`, add your `.env` (see the course setup), and run `npm install` in `ts/`.

`solution/` is a finished version to compare with when you're done. `tests/` holds the self-check.

## What you build

```
quiz-forge/
  config.toml          the pinned model, the library path, the limits
  pricing.toml         dollars per million tokens, with the date you checked them
  quizforge/
    __main__.py        python -m quizforge ...
    cli.py             commands: generate, model, cost
    config.py          reads config.toml
    library.py         lists and loads lessons from the library's MANIFEST.json
    usage.py           cost from usage, the worst case before sending, the JSONL log
    client.py          the client, and a stream that resumes when it breaks
    generate.py        one lesson (streamed) or several (async, bounded)
  tests/               copied from labs/L1/tests
  ts/generate.ts       the same call in TypeScript
```

The rules that carry over from the module: model IDs live in `config.toml`, never inline; count before you send; every request is logged; prices are dated; the key is read from `.env` and never printed.

## Steps

### 1. Settings and prices (Lessons 1.6, 1.8)
Create `config.toml` with four tables:
- `[model] writer`: the pinned snapshot you tested, `claude-haiku-4-5-20251001`.
- `[library] path`: `library/ccao-f/v1`.
- `[limits]`: `max_tokens` (1500), `max_request_usd` (0.02), `concurrency` (3).
- `[generate]`: `questions` (2), `usage_log` (`runs/usage.jsonl`), `output_dir` (`runs/items`).

Create `pricing.toml` with `source`, `checked`, and one `[models."<id>"]` table per model: `input`, `output` (dollars per million tokens), `cache_write` and `cache_read` (times the input price). Take today's numbers from the pricing page.

`quizforge/config.py` reads `config.toml` with `tomllib` (in the standard library since Python 3.11) into a frozen dataclass, with paths resolved from the project folder.

### 2. The library (Lesson 1.1's library)
`quizforge/library.py`: `lesson_ids(library)` reads `MANIFEST.json` and returns the lesson IDs; `load_lesson(library, lesson)` returns the notes, and raises `ValueError` for an unknown ID, naming the ones that exist.

### 3. Usage and cost (Lesson 1.8)
`quizforge/usage.py`:
- `cost(prices, model, usage)`: input, cache writes and cache reads at their multipliers, plus output; the cache fields can be `None`.
- `worst_case(prices, model, counted, max_tokens)`: the most a request can cost before it's sent.
- `log(path, **row)`: one JSON line per request, with every field in `FIELDS` (`time`, `label`, `model`, `request_id`, `counted`, `input`, `cache_write`, `cache_read`, `output`, `cost`, `seconds`); a row with a missing field is refused.
- `summary(path)`: requests, total cost, cost by lesson.

### 4. A stream that survives a drop (Lesson 1.5)
`quizforge/client.py`:
- `make_client(asynchronous=False)`: loads `.env` and returns `Anthropic()` or `AsyncAnthropic()`.
- `continuation(model, messages, partial)`: the request that picks up where the text stopped. Claude 4.5 models and earlier: the partial reply, with trailing whitespace removed, as the start of the assistant turn. Newer models reject that prefill, so they get a user message: "Your previous response was interrupted and ended with [the partial reply]. Continue from where you left off."
- `stream_reply(...)`: streams with `client.messages.stream`, passes each chunk to `on_text`, and on `anthropic.APIConnectionError` or `ConnectionError` keeps the text and resumes, up to `max_resumes` times. It returns the text and, for each request that finished, its final message and `stream.request_id`.

### 5. Generate (Lessons 1.3, 1.4, 1.5, 1.8)
`quizforge/generate.py`:
- One lesson: build the request (a short system prompt and the lesson plus the ask), `count_tokens` on the same request, refuse it if `worst_case` is over `max_request_usd`, stream it, log each finished request, save the text to `runs/items/<lesson>.md`.
- Several lessons: the same with `AsyncAnthropic`, an `asyncio.Semaphore(concurrency)` and `asyncio.gather`; print a line when each lesson starts and finishes, then the total, the time spent waiting, and the wall time.

### 6. The command line (Lesson 1.6)
`quizforge/cli.py` with `argparse`, run as `python -m quizforge`:
- `model`: `models.retrieve` on the pinned ID; print its lifecycle and retirement date; stop on `retired`, warn on `deprecated`.
- `generate --lesson L1.3` (repeat `--lesson` for several): checks the model first, then generates.
- `cost`: the summary of `runs/usage.jsonl`.

### 7. The same call in TypeScript (Lesson 1.3)
Rewrite `ts/generate.ts`: read the model, the limits and the library path from `../config.toml` and this model's prices from `../pricing.toml`, count the tokens, check the limit, stream with `client.messages.stream(...)`, then print the usage, the cost, the stop reason and the request ID. Run it from `ts/` with `node --env-file=../.env generate.ts L1.4`.

### 8. Check your work
1. Copy `labs/L1/tests/` into your project as `tests/`, add the pinned test packages from `solution/requirements.txt`, and run:
   ```bash
   python -m pytest -q
   ```
   All 18 tests must pass. They need no API key: the stream tests use a fake stream that drops the connection partway.
2. Then the real thing, with the environment active:
   ```bash
   python -m quizforge model
   python -m quizforge generate --lesson L1.3
   python -m quizforge generate --lesson L2.1 --lesson L2.2 --lesson L2.3 --lesson L3.1
   python -m quizforge cost
   ```
3. Commit: `git add .` and `git commit -m "Quiz Forge v0: stream questions, log usage and cost"`. `runs/` stays out of git.

## What the reference solution printed (2026-10-08)

```
$ python -m quizforge model
claude-haiku-4-5-20251001 | active | retires not scheduled

$ python -m quizforge generate --lesson L1.3
claude-haiku-4-5-20251001 | active | retires not scheduled
# Practice Questions: Lesson 1.3
...
4418 counted, 4418 in, 385 out, $0.00634, 5.43 s, request req_...

$ python -m quizforge generate --lesson L2.1 --lesson L2.2 --lesson L2.3 --lesson L3.1
claude-haiku-4-5-20251001 | active | retires not scheduled
  0.38 s  start  L2.1
  0.41 s  start  L2.2
  0.41 s  start  L2.3
  5.64 s  done   L2.2  4073 in 341 out $0.00578  end_turn
  5.94 s  start  L3.1
  6.35 s  done   L2.1  3830 in 405 out $0.00585  end_turn
  6.41 s  done   L2.3  4397 in 393 out $0.00636  end_turn
 11.68 s  done   L3.1  4268 in 364 out $0.00609  end_turn

4 lessons, $0.02408, 22.94 s of waiting, 11.68 s on the clock

$ python -m quizforge cost
5 requests, $0.03043
```

`node --env-file=../.env generate.ts L1.4` ended with `4267 counted, 4267 in, 409 out, $0.00631, end_turn` and the request ID.

Your questions, token counts and timings will differ. What should hold: the counted tokens equal the billed input, three lessons start at once and the fourth starts when one finishes, the wall time is well under the total waiting, and every request has a line in `runs/usage.jsonl` with its request ID. The whole lab costs about four cents on Claude Haiku 4.5.

## Going further (optional)
- Point `writer` at `claude-haiku-5-5`, add its prices, and compare the cost of the same lesson; then read why the counts differ (Lesson 1.1).
- Break a stream for real: unplug the network partway through a long reply, and watch it resume.

## Sources (checked 2026-10-08)
- Streaming messages (event flow, error recovery): https://platform.claude.com/docs/en/build-with-claude/streaming
- Python SDK (async client, retries, request IDs): https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/python
- Token counting: https://platform.claude.com/docs/en/build-with-claude/token-counting
- Pricing: https://platform.claude.com/docs/en/about-claude/pricing
- Models API, retrieve a model (`lifecycle`, `retires_at`): https://platform.claude.com/docs/en/api/models/retrieve
