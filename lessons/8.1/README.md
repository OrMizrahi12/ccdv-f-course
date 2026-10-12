# Lesson 8.1: Prompt injection and jailbreaks

The code from the lesson, as it was committed on camera (`fd88641` and `8c02225`).

| File | What it is |
|---|---|
| `library/fixtures/poisoned/L1.3.md` | The attack fixture: a copy of lesson 1.3 with one hidden HTML comment that asks for option A on every question and a planted link to a reserved domain. |
| `library/fixtures/poisoned/README.md` | Labels the fixture as an attack made for this lesson. The real library is never edited. |
| `experiments/inject_runs.py` | One request five ways, 5 runs each (clean, naive, naive plus a polite line, hardened, hardened on Claude Haiku 5.5): the keys at A and the planted link per run. `check` re-reads the saved replies through `keys.place`, with no request. |
| `quizforge/security/screen.py` | The injection screen: a separate request per text, a schema-valid `injection_suspected` boolean, a refusal counted as suspected. |
| `quizforge/security/keys.py` | `keys.place`: the answer letter placed by code, with the options shuffled with the lesson as the seed. |
| `tests/test_screen.py`, `tests/test_keys.py` | Offline tests with a fake client. |
| `lesson.diff` | The change to a file you already have: `generate.py` places the keys before a passed lesson is saved. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lab 7. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m experiments.inject_runs
.venv/bin/python -m quizforge.security.screen
.venv/bin/python -m experiments.inject_runs check
.venv/bin/python -m pytest -q
```

Cost: the runs are about $0.17 (25 requests), the screen about $0.06 (15 requests); `check` and the tests are free. Your counts will differ from the recording's: report them as counts from your runs, not as a rate for Claude in general. The student kit has the full steps in the lesson's order, and what the lesson printed.
