# Lesson 8.3: Guardrails and safe deployment

The code from the lesson, as it was committed on camera (`4074423`, `93f6021` and `d59f614`).

| File | What it is |
|---|---|
| `quizforge/guards.py` | Two switchable layers in the writer agent: every search result screened before the writer reads it (input), the answer letters of every saved draft placed by code (output). |
| `experiments/layers.py` | The writer agent on the poisoned L1.3 with no layer, output only, input only and both, 3 jobs each, in a scratch library and bank. |
| `quizforge/security/policy.toml` | Every agent's tools, and why. Anything not listed is denied. |
| `tests/test_policy.py`, `tests/test_guards.py` | The real tool lists must match the policy; each layer tested with a fake client. |
| `docs/threat-model.md` | What we protect, the ways in, who controls each, what an attacker wants, and the layers that answer it. |
| `docs/content-policy.md` | What Quiz Forge writes and never does, how Anthropic's Usage Policy applies, and what a refusal does. |
| `lesson.diff` | The changes to files you already have: `[guards]` in `config.toml`, the two layers in `write.py`, and the `no_screen` fixture in `tests/test_write.py`. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 8.2. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m pytest -q
.venv/bin/python -m experiments.layers
```

Cost: the layer runs are about $0.69 (12 agent jobs, about 7 minutes); the tests are free. Three runs per setting show the mechanism, not a rate. The student kit has the full steps in the lesson's order, and what the lesson printed.
