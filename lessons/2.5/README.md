# Lesson 2.5: Trust but verify

The lesson's files, exactly as they were typed on camera (commit `Rules in validate.py, a repair loop with a retry limit, and tests for both`).

| File | What it is |
|---|---|
| `quizforge/validate.py` | `loads_lenient(text)`: a defensive parse that keeps only the JSON object, whatever wraps it (a code fence, a sentence). `words(text)`: the words only, in lower case, so a quote is checked word for word, not by punctuation. `problems(data, lesson, count)`: the rules the schema can't carry: the question count, four different options, a quote found word for word in the lesson, a scenario that ends with a question. An empty list means the questions pass |
| `experiments/recheck.py` | Runs Lesson 2.4's saved replies (`ask`, `fixed`, `tool`) through `json.loads` and the rules. No API calls |
| `quizforge/repair.py` | `write_checked(...)`: one request with JSON outputs (`output_config`), `stop_reason` checked first (a refusal raises `Refused`, a cut-off reply raises `RepairFailed`), then the schema and the rules; on problems, the reply and the reasons go back to Claude, at most 2 retries, then `RepairFailed` with the reasons |
| `experiments/verify.py` | N runs of `write_checked` on Claude Haiku 4.5 (two questions about lesson L2.1), an optional `max_tokens`; prints every attempt and the totals: first try, repaired, failed |
| `tests/test_validate.py` | Offline tests: each rule rejects the right thing, a fence is stripped, and, with a scripted stand-in client, a retry repairs, the loop stops after 3 calls, and a refusal isn't retried |

Run it from your `quiz-forge` project (Lesson 2.4), with the environment active:

```bash
python -m experiments.recheck                   # Lesson 2.4's saved replies through the rules (no API calls)
python -m experiments.verify 5                   # the repair loop, five runs
python -m experiments.verify 1 300               # a reply cut off at 300 tokens fails loudly
python -m pytest -q tests/test_validate.py       # offline tests
```

About ten Claude Haiku 4.5 requests (more when replies need repairs), each with a lesson of about 4,000 tokens: about ten cents. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
