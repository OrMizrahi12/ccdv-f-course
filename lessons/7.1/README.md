# Lesson 7.1: Errors that will happen

The code from the lesson, as it was committed on camera (`2ce0f49` and `e1a4c0d`).

| File | What it is |
|---|---|
| `experiments/mock_api.py` | A local stand-in for the Messages API on 127.0.0.1:8787. It plays a script of statuses, one per arriving request (`429:2 529 200`, `spendcap`, `400`, `refusal`, `slow:30`…), and prints each arrival. |
| `experiments/retry_demo.py` | One request through the real SDK against the mock, with a dummy key. `--retries` and `--timeout` change the client options. |
| `quizforge/retry.py` | `Decision(action, reason)`, `policy(feature)` from `config.toml`, and `classify(failure)`: wait, stop, fix, fallback, continue or done. |
| `tests/test_retry.py` | Builds each SDK error from a fake response (no network, no key) and checks the decision. |
| `lesson.diff` | The changes to files you already have: the `[retry.*]` tables in `config.toml`, the `generate` policy and decision in `generate.py`, the "next:" note in `cli.py`. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the Lab 6 state, with port 8787 free. Copy the files in, then:

```bash
.venv/bin/python -m experiments.retry_demo 429:2 529 200
.venv/bin/python -m experiments.retry_demo spendcap
.venv/bin/python -m experiments.retry_demo --timeout 5 slow:30
git apply lesson.diff
.venv/bin/python -m pytest -q
.venv/bin/python -m quizforge generate --lesson L0.1
```

Cost: everything but the last command runs against the local mock with no key; the real `generate` run is one Claude Haiku 4.5 request, under half a cent. The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
