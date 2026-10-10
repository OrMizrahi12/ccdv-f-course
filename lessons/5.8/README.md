# Lesson 5.8: One app, many interfaces

The two experiments and the session test from the lesson, exactly as they were committed on camera (commit `ce82fcc`, "One question, six interfaces; one session per item, enforced").

| File | What it is |
|---|---|
| `experiments/interfaces.py` | one question, six paths: the Messages API with and without CLAUDE.md, the Agent SDK with and without project settings, `claude -p` with and without `--bare`; one table |
| `experiments/session_hygiene.py` | four reviews in one shared session vs four fresh `query()` calls: context at the start of each review, cost, and any mention of another item |
| `tests/test_sessions.py` | `ask()` must refuse a saved session that belongs to another item |

Start from your `quiz-forge` project at the Lesson 5.7 state, with the environment active and an API key in `.env`:

```bash
python experiments/interfaces.py                  # no / yes / no / yes / no / yes on camera
python -m experiments.session_hygiene             # a few minutes
python -m pytest -q tests/test_sessions.py        # 1 failed, until you add the two lines to ask()
```

Cost: on camera, one run of the six paths cost about $0.01 to $0.02 and the hygiene run about $0.29, on Haiku 4.5. The student kit has the two lines for `ask()`, the full steps for Windows, macOS and Linux, and what the lesson printed.
