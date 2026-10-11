# Lesson 7.2: Tracing and debugging

The code from the lesson, as it was committed on camera (`7446071` and `88f727b`).

| File | What it is |
|---|---|
| `quizforge/tracing.py` | OpenTelemetry for Quiz Forge: one JSON-lines trace file per run in `runs/traces/`; an SDK middleware makes a span per HTTP attempt; `Traced` wraps the client for a span per Claude call; `traced` and `validated` add the job and check spans. Spans hold ids, models, statuses and token counts, never a prompt, a reply or a key. |
| `experiments/trace_view.py` | A terminal view of one trace: the span tree, with the attributes that say where a job broke. |
| `experiments/two_layers.py` | The same job failing in two layers: `integration` (the Lesson 7.1 mock answers 529) and `output` (one real request with `max_tokens` set to 300 on purpose). |
| `tests/test_tracing.py` | Spans in memory: one scripted generate run produces the job, call and validate spans. No network. |
| `lesson.diff` | The changes to files you already have: `generate.py`, `repair.py`, `cli.py` and two new pins in `requirements.txt`. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 7.1, with `experiments/mock_api.py` and port 8787 free. Copy the files in, then:

```bash
.venv/bin/python -m pip install opentelemetry-sdk==1.45.1
git apply lesson.diff
.venv/bin/python -m experiments.two_layers integration
.venv/bin/python -m experiments.trace_view
.venv/bin/python -m experiments.two_layers output
.venv/bin/python -m quizforge generate --lesson L1.3
.venv/bin/python -m pytest -q
```

Cost: the `integration` run uses the local mock with no key; `output` and the real `generate` are Claude Haiku 4.5 requests, about two cents together. The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
