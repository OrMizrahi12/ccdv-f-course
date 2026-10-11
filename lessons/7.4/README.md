# Lesson 7.4: Testing layers

The code and data from the lesson, as they were committed on camera (`99c2917`, `0156f87` and `88a97e9`).

| File | What it is |
|---|---|
| `pytest.ini` | Registers the `integration` and `e2e` markers and leaves them out of the default run. |
| `tests/test_mcp_contract.py`, `tests/snapshots/mcp_tools.json` | The MCP server's tools (names, parameters, required parameters) held against a snapshot. Rewrite the snapshot on purpose with `pytest --update-snapshot`. |
| `tests/test_integration.py` | Marked `integration`: one real API call (request id and meter checked) and the MCP server started as a real process. Skipped without `ANTHROPIC_API_KEY`. |
| `tests/test_e2e.py`, `tests/fixtures/reply_L0.1.json` | Marked `e2e`: `python -m quizforge generate --lesson L0.1` in a fresh copy of the app, against the local mock. No key, no cost. |
| `evals/regression.jsonl` | The four fixed lessons the gate runs on. |
| `.github/workflows/evals.yml` | The regression gate in CI: `unit`, then `gate`, on pull requests that touch prompts, code, evals or settings. It needs the `ANTHROPIC_API_KEY` repository secret from Lesson 5.6. |
| `experiments/load_test.py` | 80 small requests, N at a time, against the mock's token-bucket rate limit. |
| `lesson.diff` | The changes to files you already have: the network guard and `--update-snapshot` in `tests/conftest.py`; the pinned `max_tokens` in two budget tests; `--cases`, `--gate`, `--max-usd` and the job summary in `evals/run.py`; the judge's `max_tokens`; the mock's token count, saved reply, Models API, rate limit and latency. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 7.3. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m pytest -q
.venv/bin/python -m pytest -m integration -q
.venv/bin/python -m pytest -m e2e -q
.venv/bin/python -m evals.run --cases regression.jsonl --gate --max-usd 0.10
.venv/bin/python -m experiments.load_test
```

The end-to-end test and the load test start the local mock themselves. Cost: the integration test is under $0.001 and a gate run about $0.07 (Claude Haiku 4.5, with the Lesson 7.3 judge). The load test never calls the API. The student kit has the full steps in the lesson's order, and what the lesson printed.
