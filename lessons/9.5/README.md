# Lesson 9.5: Versioning what ships

Claude Certified Developer – Foundations (CCDV-F) · Module 9: Shipping & Reuse · Lesson 5

This folder holds the files from the lesson, exactly as they were on `main` on the recording machine on 2026-10-12, after both pull requests were merged. It also has the steps to repeat the lesson in your own `quiz-forge` project, starting from the end of Lesson 9.4. The commits:

- `cca9e3b` "Release 0.9.0: a manifest of what ships, a model lock and a changelog"
- `d0bdc68` "quiz-forge-tools 1.1.0: the guard hook from 8.4"
- `2bc055c` "The writer's upgrade evidence: write jobs graded by code and the judge, the rule written first"
- `12423cd` "Writer on claude-haiku-5-5: the migration guide's list applied"
- `73daba4` "The writer on the new model: our code fixed where the evidence pointed"
- `e11cee4` "Release 0.10.0: the writer on claude-haiku-5-5, its evidence in the changelog, the upgrade runbook"
- two merge commits on `main`: `a8cfb81` (pull request 14, release 0.9.0) and `cd761a3` (pull request 15, the upgrade)

Tags: `v0.9.0`, `quiz-forge-tools--v1.1.0` and `v0.10.0`.

By the end of the lesson you can:

- pin models, and version prompts and configs;
- manage plugin dependencies with version ranges;
- upgrade a model with evals, ship it through a pull request, and keep a one-commit rollback.

**Quick check:** before you move on, take this lesson's quick check. It's a few short questions, and it takes a couple of minutes.

## What's in this folder

| File | What it is |
|---|---|
| `models.lock.toml` | Each model ID `config.toml` uses, with its status and the earliest date it could retire, copied from the model deprecations page, with the day it was checked. Re-check it monthly. |
| `quizforge/release.py` | The release manifest: the version, the commit, a hash of `config.toml`, each feature's model (checked against the Models API and the lock), and each prompt's locked version. `--check` fails on an alias, on a model missing from the lock, or on a model the API calls deprecated. |
| `CHANGELOG.md` | The versioning rules (MAJOR, MINOR, PATCH; a model change is at least MINOR, with its evidence), then the 0.9.0 and 0.10.0 entries. |
| `tests/test_release.py` | Offline tests for the manifest, against a stand-in Models API: an alias, a model missing from the lock, and a deprecated model are each a problem. |
| `evals/writer.py` | The upgrade's evidence: the same write jobs on the regression lessons with one model, into a copy of the bank. Each final draft is graded by code and by the calibrated judge, which is held constant. `--against <baseline>` compares the saved runs, applies the rule, and lists every job that didn't finish, with its reason. |
| `tests/test_upgrade.py` | The upgrade rule, written before either run: no rate may fall more than the allowance below the baseline, and a done lesson must stay within the write job's budget. |
| `lesson.diff` | The changes to files that already existed: `__version__`, `config.toml` (the writer's model, `[route.batch]`, `[write] max_tokens`), `config.py`, `cli.py`, `sources.py`, `write.py`, `agent.py`, their tests, the plugin's version in `plugin.json` and `marketplace.json`, and the upgrade section of `docs/runbook.md`. Apply it with `git apply lesson.diff`, or make the changes yourself as the lesson does. |

## Before you start

- Your `quiz-forge` project at the end of Lesson 9.4, on `main`, with a clean `git status`, pushed to your own GitHub repository with the CI from Module 6.
- Your API key in `.env`.
- Claude Code, for the migration and the fixes.
- Cost: `release` and `release --check` read the Models API, which costs nothing. Each writer eval run is four write jobs, so expect a few cents per run. On the recording machine, a write job cost about $0.05 per lesson on the baseline model, and about half a cent on the new one.

## Steps (the lesson's order)

1. **What exactly ships:**
   1. Make a release branch: `git switch -c release/0.9.0`.
   2. Find every pin: `grep -n "claude-" config.toml`. Then search the code too, to check that no model ID is hard-coded.
   3. Add `models.lock.toml`, with the dates from the deprecations page on the day you check it.
   4. Add `quizforge/release.py`, and run `.venv/bin/python -m quizforge.release`.
   5. Set `__version__` to `0.9.0`, add `CHANGELOG.md` and `tests/test_release.py`, then run the tests.
   6. Commit, and run `.venv/bin/python -m quizforge.release --check`. It should exit 0.
2. **The plugin ships too:**
   1. List the plugin's tags, and see what changed since the last one: `git diff --stat quiz-forge-tools--v1.0.0 -- plugin/quiz-forge-tools`.
   2. A new hook is a new feature: set `"version": "1.1.0"` in `plugin.json`. Leave the `qf-format` dependency's range at `^1.0`.
   3. Run `claude plugin validate --strict plugin/quiz-forge-tools`, then `claude plugin tag --dry-run plugin/quiz-forge-tools`. The dry run reports the marketplace's mismatch.
   4. Set the same version in `marketplace.json`, validate again, commit, and repeat the dry run.
3. **The writer's upgrade, with evidence:**
   1. Add `evals/writer.py` and `tests/test_upgrade.py`. Run the tests, and commit.
   2. Run the baseline: `.venv/bin/python -m evals.writer --model claude-haiku-4-5-20251001`.
   3. Make a branch, `upgrade/writer-haiku-5-5`, start `claude`, and ask Claude Code's Claude API skill to migrate the writer only. Approve each edit after you've read it.
   4. Read the diff, run the tests, and commit.
   5. Run the candidate: `.venv/bin/python -m evals.writer --model claude-haiku-5-5 --against claude-haiku-4-5-20251001`.
4. **Fix, decide, ship:**
   1. If the rule says hold, read why: `.venv/bin/python -m evals.writer --against claude-haiku-4-5-20251001` (free; it reads the saved runs).
   2. Fix what's in your own code, with Claude Code, one reviewed edit at a time. Then run the tests, commit, and run the candidate again.
   3. When the rule says ship: add the upgrade section to `docs/runbook.md`, add a `0.10.0` entry to `CHANGELOG.md` with both models' numbers, set `__version__` to `0.10.0`, then run the tests and commit.
   4. Push both branches, and open two pull requests: first `release/0.9.0`, then the upgrade. Wait for the checks, then merge each with a merge commit (`gh pr merge --merge`).
   5. On `main`: `git pull`. Then tag each release where it shipped: `v0.9.0` on the release branch's last commit, `claude plugin tag plugin/quiz-forge-tools`, and `v0.10.0` on `main`. Push the tags.
   6. Run `.venv/bin/python -m quizforge.release --check` one last time.

On Windows, the venv's Python is `.venv\Scripts\python`.

## What the lesson printed

These results are from the recording machine (2026-10-12). Model output varies, so your numbers will differ, and a held run may stop for other reasons. The method is what to copy: read why each job didn't finish, and fix what's yours.

**The first manifest:** `quiz-forge 0.1.0 at aabdddb`. All four models were active, but the writer's `claude-haiku-4-5-20251001` said `watch`: it could retire as soon as 2026-10-15, inside the 60 days of notice.

**The plugin's dry run, before the marketplace fix:** a version mismatch: `plugin.json` says `"1.1.0"`, but the marketplace entry still says `"1.0.0"`. `plugin.json` wins at install time. After the fix: `would create tag quiz-forge-tools--v1.1.0`.

**The writer evals:**

| Run | Done | Code | Supported | Judge: good | $ per done lesson | p95 |
|---|---|---|---|---|---|---|
| Baseline, `claude-haiku-4-5-20251001` | 100% | 88% | 100% | 62% | 0.0509 | 53.1 s |
| Candidate, first run: **hold** | 25% | 75% | 50% | 50% | 0.0255 | 66.5 s |
| Candidate, after the fixes: **ship** | 100% | 75% | 100% | 62% | 0.0047 | 33.7 s |

On the first candidate run, three jobs didn't finish. All three causes were in our own code, not in the model:
- **An API error:** a tool error was sent with empty content.
- **The rounds limit:** the checker now adds a reason after its verdict, and our parser missed it.
- **Cut off:** Haiku 5.5's thinking counts toward `max_tokens`, and the writer shared a limit of 1500.

**The fixes:** 9 files changed, 59 insertions, 7 deletions; `323 passed, 4 deselected`.

**The pull requests:** 4 checks on each, all successful. Both were merged with a merge commit.

**The last release check:** `quiz-forge 0.10.0 at cd761a3`:
- the writer on `claude-haiku-5-5`, active;
- batch still pinned to `claude-haiku-4-5-20251001`: active, `watch`;
- exit code 0.

Batch is the next upgrade.
