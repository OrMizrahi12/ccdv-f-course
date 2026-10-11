# Lab 7 solution notes

Read these after you've tried the lab. They explain the decisions in `solution/`, the mistakes the self-check catches, and what to compare with your own version.

## The decisions, and why
- **The gate checks its judge before it checks anything else.** A judge is only as good as its last calibration. If the judge's model, rubric version or rubric text changed since `calibration.json` was written, its verdicts are unmeasured, so the gate refuses to run rather than produce a confident number nobody can trust. The check reads two files and costs nothing.
- **The rubric's hash, not just its version name.** The hash comes from `prompts/judge/LOCK`, the same lock `generate`'s prompts use. Editing a locked rubric in place would keep the name and change the judge; the hash catches it.
- **A judge below its line still gates the rest.** Our judge (Claude Sonnet 5.5, rubric v2) reached 80% agreement, kappa 0.62 and 2 false passes on the held-out labels, below the 90% / 0.7 / 1 in `criteria.md`. Gating quality on it would block or pass pull requests on noise: two gate runs on the same code scored 3 of 8 and then 8 of 8 items good. So quality is reported as `below`, and completion, quotes, cost and latency, which code measures, are gated. The day a rubric reaches the line on held-out, quality starts gating with no code change.
- **Thresholds live in two places on purpose, with a test between them.** People read `criteria.md`; code reads `config.toml`. A test fails when they disagree, so neither can be edited alone.
- **A baseline and an allowance, not just thresholds.** A threshold catches a feature that is bad; the baseline catches one that got worse. With four regression lessons, one lesson is 25% of completion, so the allowance is one lesson: a single wobble passes, two lessons lost fail.
- **`--accept` is a person's command.** If the gate could rewrite its own baseline, a slow decline would be accepted one pull request at a time. The command refuses to run where `GITHUB_ACTIONS` is set.
- **`evaluate()` and `calibrate()` are functions.** The CLI, the tests and the old scripts (`python -m evals.run`, `python -m evals.calibrate`) share them. The tests replace `evaluate` with a fake and check the gate's decisions without a single request.
- **Offline by default.** The default `pytest` run has a network guard, so a test that forgot its fake fails loudly instead of quietly spending money. Live tests are marked and run on purpose.

## Mistakes the self-check catches
| Symptom | Cause | Test that fails |
|---|---|---|
| The gate runs with a changed judge | no freshness check, or a check on the version name only | `test_lab7_gate.py::test_stale_calibration_refused` |
| The gate spends money and then refuses | the freshness check comes after the first request | `test_lab7_cli.py::test_gate_refuses_a_stale_judge_before_any_request` |
| A pull request fails on a judge nobody trusts | quality gated with an uncalibrated judge | `test_lab7_gate.py::test_quality_is_gated_only_with_a_trusted_judge` |
| `criteria.md` says 80% and the gate uses 70% | thresholds edited in one place only | `test_lab7_gate.py::test_config_matches_criteria` |
| A slow decline is never caught | no baseline comparison | `test_lab7_cli.py::test_accept_then_a_drop_fails` |
| CI moves its own baseline | `--accept` runs anywhere | `test_lab7_gate.py::test_accept_is_never_run_by_ci` |
| A tuning run on dev unlocks the gate | `--calibrate --split dev` writes `calibration.json` | `test_lab7_cli.py::test_calibrating_on_dev_records_nothing` |
| The report stays green over budget | cost counted per request instead of per lesson, or p95 not checked | `test_lab7_cli.py::test_report_marks_each_breach` |

## What to compare
- Your held-out agreement, kappa and false passes against your `criteria.md` lines. If the judge only reached them after you tuned it on held-out items, it hasn't reached them.
- The gate's spend per run against `--max-usd`. Ours spent $0.039 and $0.042 of a $0.10 cap; nearly all of it was the judge.
- `solution/evals/calibration.json` and `baseline.json` are from our runs. Yours should come from your own judge and your own accepted run.
