# Lesson 9.3: Refactoring and modernization with Claude

Claude Certified Developer – Foundations (CCDV-F) · Module 9: Shipping & Reuse · Lesson 3

This folder holds the files from the lesson, exactly as they were committed on the recording machine on 2026-10-12. It also has the steps to repeat the lesson in your own `quiz-forge` project, starting from the end of Lesson 9.2. The commits:

- `699ecf0` "One client factory: each feature's options set in one place, the contract tested first"
- `d2ca152` "A dependency map, a staged migration plan and a modernization scope"

By the end of the lesson you can:

- refactor safely, with tests written first;
- plan a large migration in stages;
- scope a code modernization engagement.

**Quick check:** before you move on, take this lesson's quick check. It's a few short questions, and it takes a couple of minutes.

## What's in this folder

| File | What it is |
|---|---|
| `tests/test_client_contract.py` | The contract, written before the refactor: what each path's client must keep. Generate's policy (4 retries, a 120 s timeout, the tracing middleware) for one lesson and for many; the SDK defaults for every other feature; the base URL from the environment. No request is sent. |
| `lesson.diff` | The refactor Claude Code made, with every edit approved: `make_client(feature)` in `quizforge/client.py`, `options(feature)` in `quizforge/retry.py`, and both generate paths in `quizforge/generate.py` asking the factory for generate's client. Apply it with `git apply lesson.diff`, or ask Claude Code for the same refactor (step 2). |
| `experiments/depmap.py` | Quiz Forge's own import graph, as levels: level 0 imports no other `quizforge` module, and each level imports only from the levels before it. |
| `docs/migration-plan.md` | A staged plan for porting the core and CLI to TypeScript: a planning exercise, not scheduled. The judge first, then the rulebook, the order, gaps, the loop, gates, and cost. |
| `docs/modernization-scope.md` | A scope for a hypothetical client engagement (an invented regional insurer's legacy claims service). |

## Before you start

- Your `quiz-forge` project at the end of Lesson 9.2, on `main`, with a clean `git status`.
- Claude Code, signed in.
- Your API key in `.env`, for the one smoke-test request at the end of step 3.
- Cost: one `generate` for one lesson (about $0.002 on the recording machine). Claude Code runs on your Claude plan or API key.

## Steps (the lesson's order)

1. **Find the target, and pin the behavior:**
   1. Find where clients are made, and where their options are set: `grep -rn "make_client(\|with_options(" quizforge evals`.
   2. Add `tests/test_client_contract.py` before you change anything.
   3. Run it on the old code: `.venv/bin/python -m pytest -q tests/test_client_contract.py`. On the recording machine, three passed and one failed: several lessons at once got a plain client, with no retry policy and no tracing.
2. **The refactor, with Claude Code:**
   1. Start Claude Code in plan mode: `claude --permission-mode plan`.
   2. Brief it: one `make_client(feature)` in `quizforge/client.py` sets that feature's options (its `[retry.<feature>]` policy when `config.toml` has one, and the tracing middleware), so no caller sets client options itself. Keep the line that builds the base client. The contract file is not Claude's to change, and it must go green. Plan first, then one call site at a time, running `pytest -q` after each. Make each change with the Edit tool. Don't commit.
   3. Read the plan, including what it leaves alone and why. Accept it with "Yes, manually approve edits", and read each edit before you approve it.
3. **Read it, commit it, try it:**
   1. Exit Claude Code, and run the whole suite yourself: `.venv/bin/python -m pytest -q`.
   2. Read the change: `git --no-pager diff --stat`, then commit.
   3. Run one real request as a smoke test: `.venv/bin/python -m quizforge generate --lesson L1.3`, then `tail -1 runs/usage.jsonl`.
4. **Plan big, without moving any code:**
   1. Add `experiments/depmap.py`, and run it: `.venv/bin/python -m experiments.depmap`.
   2. Add `docs/migration-plan.md` and `docs/modernization-scope.md`, then commit.

On Windows, the venv's Python is `.venv\Scripts\python`.

## What the lesson printed

These results are from the recording machine (2026-10-12). Your test count, costs and levels can differ.

**The contract, on the old code:**

```
FAILED tests/test_client_contract.py::test_many_lessons_get_the_generate_policy - assert False
1 failed, 3 passed in 0.70s
```

**During the refactor:** still 1 failed (the many-lessons test, as expected) after the factory and after `generate_one`; all green after `generate_many`.

**After the refactor:**

```
291 passed, 4 deselected in 3.33s
 quizforge/client.py   | 11 +++++++++--
 quizforge/generate.py |  4 ++--
 quizforge/retry.py    |  6 ++++++
 3 files changed, 17 insertions(+), 4 deletions(-)
```

**The smoke test:**

```
L1.3: passed after 1 attempt(s), $0.00156 -> runs/items/L1.3.json
```

**The dependency map:**

```
level 0: quizforge, budget, config, library, prompts, schema, security.guard, security.keys, security.redact, usage
level 1: agent, files, library_client, meter, report, retrieve, retry, sanitize, security.integrity, security.secret_scan, tools, tracing, validate
level 2: ask, client, evidence, export, memory, repair, search, sources
level 3: generate, review, security.screen, whoami, writer_tools
level 4: batch, guards, team
level 5: write
level 6: cli
level 7: __main__
```

**What Claude Code left for you to decide:** the `[retry.check]` settings, which nothing uses; experiments that build their own clients for a mock server; and an async client with a feature (the tracing middleware is sync only).
