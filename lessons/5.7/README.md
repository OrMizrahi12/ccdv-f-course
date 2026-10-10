# Lesson 5.7: The Claude Agent SDK

The Agent SDK reviewer from the lesson, exactly as it was committed on camera (commit `c7c3361`, "An Agent SDK reviewer: query() with the plugin's skill, read-only tools, a scope hook, resumable sessions"; `qf review` then shows its verdict, commit `833d8cb`).

| File | What it is |
|---|---|
| `reviewer/sdk_review.py` | the reviewer: `Verdict`, the `stay_in_scope` hook, `options()`, `review()` (one `query()` per file), `ask()` (resume), and the command line |
| `reviewer/__init__.py` | empty: makes `reviewer` a package, so `python -m reviewer.sdk_review` works |
| `tests/test_sdk_review.py` | the hook and the `Verdict`, tested without a model call |

Add the reviewer's model and limits to `config.toml` first (the student kit has the lines: `reviewer = "claude-sonnet-5-5"` and a `[review]` table with `max_turns` and `max_usd`). Run from your `quiz-forge` project, with the environment active:

```bash
python -m pytest -q tests/test_sdk_review.py          # 3 passed on camera
python -m reviewer.sdk_review bank/pending/L1.3.json  # one review: model, skill loaded, verdict, cost, turns
python -m reviewer.sdk_review --ask L1.3 "Which option did you object to?"
```

Cost: on camera, one review cost $0.06 to $0.14 on Claude Sonnet 5.5 (12 to 16 turns). The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
