# Lesson 5.1: Operating Claude Code

The files Claude Code changed in the lesson, exactly as they were committed on camera (commit `641db08`, "Add 'quizforge bank': per-lesson counts of pending and approved questions", on top of Quiz Forge v0.4 from Lab 4).

| File | What changed |
|---|---|
| `quizforge/review.py` | `counts(cfg)`: how many questions each lesson has in `bank/pending/` and `bank/approved/`, read-only |
| `quizforge/cli.py` | `show_bank(cfg)` and a `bank` subcommand: one line per lesson, then a total |
| `tests/test_review.py` | four new tests: the counts, the empty bank, the bank left untouched, the printed format |

The lesson drives one feature through explore, plan, code and commit in Claude Code: plan mode as the gate before any edit, every edit approved by hand, a rewind with a checkpoint, `/context` and `/compact`, and `claude --continue`, `/clear` and `/resume`. Your own session will write different code; compare, don't paste.

```bash
python -m pytest -q          # 107 passed on camera (103 + 4)
python -m quizforge bank     # per-lesson counts of pending and approved questions
```

No API spend: Claude Code runs on your subscription, and the new command makes no API call. The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
