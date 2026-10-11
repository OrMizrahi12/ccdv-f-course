# Lesson 7.3: Evals

The code and data from the lesson, as they were committed on camera (`639e036`, `c806efc`, `33877a0` and `c530a63`).

| File | What it is |
|---|---|
| `evals/criteria.md` | What "done" means for `generate`, and the line the judge has to meet. Written and committed first. |
| `evals/golden.jsonl`, `evals/to_label.jsonl` | 27 reference labels as they ship (8 good, 19 bad, each with a reason, a source and a split), and the three items you label yourself. |
| `evals/graders.py` | Code graders: the schema, Quiz Forge's own rules (quotes word for word), and the long-key check added in the lesson. |
| `evals/label.py` | Labels items by hand into `golden.jsonl`. |
| `prompts/judge/v1.md`, `v2.md`, `LOCK` | The judge's rubric, before and after its one change (test each distractor; a strawman fails the item). |
| `evals/judge.py`, `evals/calibrate.py` | The LLM judge, and its calibration against the labels: agreement, Cohen's kappa, false passes and cost. |
| `evals/cases.jsonl`, `evals/run.py` | The capability eval on ten fixed lessons, held against `criteria.md`. Exit code 1 when any line fails. |
| `tests/test_evals.py` | The graders and kappa, tested offline. |
| `lesson.diff` | The changes to files you already have: `[judge]` in `config.toml`, Claude Sonnet 5.5 prices in `pricing.toml`, `--folder` for `prompts` in `cli.py`. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 7.2. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m evals.graders
.venv/bin/python -m evals.label
.venv/bin/python -m quizforge prompts --folder prompts/judge --lock v1
.venv/bin/python -m evals.calibrate --split dev --prompt v1
.venv/bin/python -m evals.run; echo "exit code $?"
.venv/bin/python -m pytest -q
```

Cost: the calibration runs come to about $0.20 (Claude Haiku 4.5, then Claude Sonnet 5.5 as the judge), and the eval about $0.13. The student kit has the full steps in the lesson's order, and what the lesson printed.
