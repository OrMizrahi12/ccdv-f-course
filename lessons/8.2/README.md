# Lesson 8.2: Protecting data

The code from the lesson, as it was committed on camera (`bfd0a2b`, `d2768a5` and `c887994`).

| File | What it is |
|---|---|
| `quizforge/security/redact.py` | Three patterns (an email address, a phone number, an API key), each replaced by a placeholder that says what was there; `found(text)` counts them. |
| `experiments/data_map.py` | Every place Quiz Forge sends or keeps text, with the emails, phones and keys in it. Counts only. |
| `library/fixtures/pii/` | The privacy fixture: lesson 2.1 plus one question from an invented reader (an `example.com` address, a 555-01xx number). |
| `experiments/minimize_check.py` | The input tokens the writer would be sent for the fixture, as written and minimized (token counting is free). |
| `quizforge/security/integrity.py` | Seals the approved bank (`MANIFEST.sha256`, the `sha256sum` format) and verifies it: a file changed, added or missing since a person's yes. |
| `tests/test_privacy.py`, `tests/test_integrity.py` | Offline tests. |
| `lesson.diff` | The changes to files you already have: `minimize()` in `sanitize.py`, `redact_obj` before every save, and the seal after a person's yes in `review.py`. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 8.1. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m experiments.data_map
.venv/bin/python -m experiments.minimize_check
.venv/bin/python -m quizforge.security.integrity seal
.venv/bin/python -m quizforge.security.integrity; echo "exit code $?"
.venv/bin/python -m pytest -q
```

Cost: none. Token counting is free, and nothing else calls the API. The student kit has the full steps in the lesson's order, and what the lesson printed.
