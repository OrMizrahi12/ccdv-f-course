# Lab 7: one release process: a proven judge, a gate in CI, a budget report

Claude Certified Developer – Foundations (CCDV-F) · Module 7: Production Engineering, Evals & Cost · the module checkpoint

In Module 7 you made Quiz Forge production-grade one concern at a time: an error class and a recovery decision for every failure, with SDK retries where they help (7.1); OpenTelemetry spans for every attempt and call, and a terminal trace viewer (7.2); success criteria written first, code graders, labeled items and a judge calibrated against the labels (7.3); unit, contract, functional, integration and end-to-end tests, a regression gate in CI and a load test against a mock (7.4); the lesson as a cached prefix, with cache-aware cost (7.5); budgets per feature, a measured route for `generate` and one ledger for several agents (7.6). In this lab they become one release process that a change has to pass:

- `qf eval --calibrate`: the judge against the human labels, recorded in `evals/calibration.json` with the exact judge it measured.
- `qf eval --gate`: refuses to run with a judge that wasn't calibrated in its current form; then holds the regression set to the criteria and to the last accepted baseline, with a spend cap. `.github/workflows/evals.yml` runs it on pull requests, after the offline tests.
- `qf report`: cost per task, p95 latency and cache reads per feature (or per model), against the budgets; it exits 1 on a breach.

You pass the lab when the self-check tests pass and one real calibration, one real gate run and one real report run finish against your own criteria.

## What you start from

`starter/` is the `quiz-forge` project exactly as it stands after Lesson 7.6: Lab 6's server and `qf ask`; `quizforge/retry.py` (7.1); `quizforge/tracing.py` and `experiments/trace_view.py` (7.2); `evals/` with `criteria.md`, `golden.jsonl` (30 labeled items: 20 dev, 10 held-out), `graders.py`, `judge.py`, `calibrate.py`, `run.py`, `cases.jsonl` and `regression.jsonl`, and the locked judge rubrics in `prompts/judge/` (7.3, 7.4); the test markers, the network guard and `evals.yml` (7.4); the cached lesson block and cache-aware cost (7.5); `docs/budgets.md`, the `[budget.*]` and `[route.generate]` tables, `quizforge/report.py` and `quizforge/budget.py` (7.6). Its 183 tests pass. If your own project is there too, keep working in it; if not, copy `starter/` to `~/projects/quiz-forge`, add your `.env`, and install the pinned `requirements.txt`.

`solution/` is a finished version to compare with when you're done; `SOLUTION-NOTES.md` explains its main decisions. `tests/` holds the self-check.

## What you build

```
quiz-forge/
  config.toml          + [eval]: the thresholds from evals/criteria.md, the judge's line, the baseline allowance, the gate's spend cap
  evals/
    gate.py            the gate: judge freshness, thresholds, baseline comparison, the Markdown summary, --accept
    calibration.json   written by qf eval --calibrate --split heldout (commit it)
    baseline.json      written by qf eval --accept, by a person (commit it)
    run.py             the eval as a function (evaluate) the gate can call
    calibrate.py       the calibration as a function that returns what it measured
  quizforge/
    cli.py             + eval (--calibrate | --gate | --accept, --split, --max-usd), report (--since, --by)
    report.py          the report as functions; --since 1d / 12h / 90m, --by feature or model
  .github/workflows/evals.yml   the gate job runs python -m quizforge eval --gate --max-usd 0.10
  docs/release.md      how a change ships: which checks run where, what blocks a merge, what only a person decides
  tests/               copied from labs/L7/tests
```

The rules that carry over from the module: retry what is transient and only that; log request ids and usage, never request bodies by default; decide what "done" means before measuring; code graders for what code can check, a judge only for the rest, and a judge is trusted only as far as its agreement with people; tests that need no network by default, and live tests marked; budgets per feature, measured per completed task; a release is gated on evidence.

## Steps

### 1. Thresholds in one place (Lessons 7.3, 7.6)
Add `[eval]` to `config.toml` with the numbers in `evals/criteria.md`: completion (9 of 10 lessons → `0.9`), quotes (`1.0`), quality (`0.8`), cost per completed lesson (`0.02`), p95 seconds (`30`), and the judge's line on held-out labels (agreement `0.9`, kappa `0.7`, at most `1` false pass). Two more: `allowance`, how far a rate may fall below the accepted baseline (the solution uses `0.25`: one lesson of the four in `regression.jsonl`), and `max_usd`, the gate's spend cap (`0.10`). A test reads both files and fails if they disagree, so the document people read and the numbers the gate uses can't drift apart.

### 2. A judge you can trust, and prove it (Lesson 7.3)
Turn `evals/calibrate.py` into a function that returns what it measured (agreement, kappa, false passes, false fails, cost), and add `qf eval --calibrate --split heldout`:
- it runs the judge from `[judge]` over the held-out labels and writes `evals/calibration.json`: the judge's model, its rubric version and that version's hash from `prompts/judge/LOCK`, the split, the numbers, and the date;
- it exits 1 when the judge misses its line, and says so. Calibrating on `dev` is for tuning and records nothing.

### 3. The gate (Lesson 7.4)
`evals/gate.py`, run by `qf eval --gate`:
- **The judge first:** if `calibration.json` is missing, isn't from `heldout`, or its model, rubric version or rubric hash differs from what `config.toml` and the LOCK say now, the gate stops before any request: "re-calibrate the judge before it can gate".
- Then the regression set, through `evaluate()`: generate each lesson with the current route, run the code graders, then the judge on what passed, stopping before `--max-usd` (default `[eval] max_usd`).
- Hold the totals to `[eval]`. Quality is gated only if the judge met its own line; until then it is reported as `below`, not failed.
- Compare each rate with `evals/baseline.json`; a fall bigger than `allowance` fails. With no baseline yet, say so and don't fail.
- Print the rows; write the same table as Markdown to `$GITHUB_STEP_SUMMARY` when it is set; save the run to `runs/last-gate.json`; exit 1 on any FAIL.
- `qf eval --accept` turns the last passing run into `baseline.json`. It refuses to run in CI (`GITHUB_ACTIONS` is set there): only a person moves the baseline.

### 4. The pipeline (Lessons 7.4, 5.6)
In `.github/workflows/evals.yml`, the `gate` job now runs `python -m quizforge eval --gate --max-usd 0.10`. Everything else stays: `on: pull_request` with the paths filter (`prompts/**`, `quizforge/**`, `evals/**`, `config.toml`, `pricing.toml`), `permissions: contents: read`, one concurrency group per ref with `cancel-in-progress: true`, `unit` (offline `pytest`) before `gate` (`needs: unit`, a timeout), and the API key from the repository secret. Commit `evals/calibration.json`: without it, CI's gate refuses. Write `docs/release.md` (one screen): which check runs where (laptop, pull request, `main`), what blocks a merge, and that merging, `--accept` and a new judge are a person's decisions.

### 5. The report (Lessons 7.5, 7.6)
Turn `quizforge/report.py` into functions and add `qf report --since 1d [--by feature|model]`: requests, cost, cost per task, p95 latency and the cache-read share per feature, each `[budget.*]` breach marked `OVER`, exit code 1 if there is any. `--since` takes `1d`, `12h` or `90m`; `--by model` shows the same columns per model, without budgets.

### 6. Check your work
1. Copy `labs/L7/tests/` into your project as `tests/` and run:
   ```bash
   python -m pytest -q
   python -m pytest -m e2e -q
   ```
   All tests must pass. The default run is offline: a guard fails any test that opens a network connection. The 24 Lab 7 tests (`test_lab7_*.py`) check that `[eval]` and `criteria.md` agree; that a calibration records the judge it measured, and that a missing, `dev`, or changed-judge calibration is refused; that quality is gated only with a trusted judge; that the criteria and a drop beyond the allowance fail the gate; that `--accept` never runs in CI; that `qf eval` refuses a stale judge before any request, writes its run, and takes the spend cap from config or `--max-usd`; that `qf report` marks each breach in a fixture log; and that the workflow and `docs/release.md` say what they should. On the starter, 22 of them fail; the other 183 tests are the module's own.
2. Then the real thing, with the environment active:
   ```bash
   python -m pytest -m integration -q
   python -m quizforge eval --calibrate --split heldout
   python -m quizforge eval --gate
   python -m quizforge eval --accept
   python -m quizforge report --since 1d
   ```
   The integration tests cost a fraction of a cent. A held-out calibration with Claude Sonnet 5.5 as the judge is 8 judge requests, about 3 cents; a gate run is about 4 cents (generation on Claude Haiku 5.5 is a tenth of a cent per lesson; the judge is most of it), and never more than the cap.
3. Try the refusal: set `[judge] prompt = "v1"` and run the gate. It should stop before any request. Set it back.
4. Optional, if your repository is on GitHub: commit `evals/calibration.json`, change something harmless in `quizforge/`, open a pull request and watch both jobs.
5. Commit: `git add .` and `git commit -m "Quiz Forge v0.7: one release process: a judge proven on held-out labels, a gate that refuses a stale judge, and a budget report"`.

## Rubric (what the tests can't see)
| Check | Pass when |
|---|---|
| Criteria came first | `git log --follow evals/criteria.md` shows it committed before the first eval result |
| The judge is earned | `calibration.json` is from `heldout`, and the judge was never tuned on those items |
| The baseline is a decision | `baseline.json` was written by `--accept`, by you, after a run you read |
| CI is cheap and bounded | The gate runs only on relevant paths, cancels superseded runs and has a spend cap |
| Budgets come from data | Every number in `docs/budgets.md` can be traced to a report or a measured run |
| Logs are safe by default | `runs/usage.jsonl` holds no prompt or reply text |

## What a passing run looks like
Our run, 2026-10-11 (your numbers will differ):
- `--calibrate --split heldout`: Claude Sonnet 5.5, rubric v2, the 2×2 table, then `agreement 80%, kappa 0.62, false passes 2; 8 judge requests $0.0344`, and "the judge misses its line": exit 1, written to `calibration.json` anyway. Our judge hasn't earned a quality gate yet, so the gate reports quality and gates the rest.
- `--gate`, first run: one row per lesson, then `PASS completion 4 of 4`, `PASS quotes 8 of 8`, `below quality 3 of 8 items judged good (38%)`, `PASS cost $0.0013 per completed lesson`, `PASS latency p95 9.3 s`, `none baseline`, and `gate: pass, $0.0391 spent`.
- `--accept`, then a second gate run on the same code: completion and quotes the same, quality 8 of 8, three `vs baseline` rows, `gate: pass, $0.0424 spent`. The same code went from 3 of 8 to 8 of 8 judged good: that is why quality isn't gated until the judge earns it.
- With the judge's rubric switched to v1: `gate: refused before any request: the judge's prompt changed since its calibration (v2 -> v1): re-calibrate the judge before it can gate`, exit 1.
- `report --since 1d`: one line per feature; ours had 241 requests, $0.97, `generate` at $0.0054 per lesson and 28% read from the cache, both budgets `ok`, exit 0.

## Going further (optional)
- Write a v3 judge rubric, lock it, tune it on `dev`, and calibrate it once on `heldout`. Does it reach the line, so the gate can start gating quality?
- Run the gate twice on the same code and set `allowance` from what you see. How far apart are two honest runs?

## Sources (as checked for Lessons 7.1 to 7.6)
- Define success criteria and build evaluations: https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
- Demystifying evals for AI agents: https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
- Errors and rate limits: https://platform.claude.com/docs/en/api/errors, https://platform.claude.com/docs/en/api/rate-limits
- Prompt caching: https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Optimizing for cost and intelligence: https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence
- GitHub Actions workflow syntax and job summaries: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax, https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands
