# Lesson 2.6: Iterating on a prompt

The lesson's files, exactly as they were typed on camera (commit `Prompt v3 and v4, measured; sanitize.py for lesson input; measure.py and failures.py`).

| File | What it is |
|---|---|
| `quizforge/sanitize.py` | `clean(text)`: a lesson is untrusted text, so before it goes inside `<lesson>` tags, drop control and invisible characters (Unicode category C, except newline and tab) and any lesson tag inside it |
| `tests/test_sanitize.py` | Offline tests: a lesson can't close its tag, invisible characters are removed, and every real lesson in the library comes out unchanged |
| `prompts/item_writer/v3.md` | The system prompt as of Lesson 2.5 (`system.md`), saved as a version: the baseline |
| `prompts/item_writer/v4.md` | v3 plus one sentence in rule 4: how the quote is checked |
| `prompts/item_writer/CHANGELOG.md` | Each version: what changed, why, and what it measured |
| `experiments/measure.py` | `python -m experiments.measure <version> <runs>`: the same three lessons (L1.3, L2.1, L3.2), the same model (Claude Haiku 4.5) and the same ask for every version; each reply checked with `problems()`; failures counted by kind; at most 6 requests at once. Replies saved to `runs/<version>-<lesson>-<run>.json`, the summary line to `runs/measure.log` |
| `experiments/failures.py` | `python -m experiments.failures <version>`: every saved quote the lesson doesn't contain, to read the failures, not just count them |

Run it from your `quiz-forge` project (Lesson 2.5), with the environment active:

```bash
python -m pytest -q tests/test_sanitize.py      # offline tests for clean()
python -m experiments.measure v3 6              # the baseline: three lessons, six runs each
python -m experiments.failures v3               # every failed quote, to read
python -m experiments.measure v4 6              # one change, measured the same way
```

36 Claude Haiku 4.5 requests (18 per version), each with a lesson of about 4,000 tokens: about 30 cents. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
