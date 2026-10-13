# Student kit: Lesson 10.1, Capstone tour

Claude Certified Developer – Foundations (CCDV-F) · Section 10: Exam Readiness · Lesson 1

This lesson adds no code. It walks your finished `quiz-forge` project (the end of Lab 9, version 1.0.0) layer by layer, then runs it once, end to end. This kit has the layer map, the commands of the run, and what to check at each step.

By the end of the lesson you can:

- walk the whole of Quiz Forge, part by part;
- name the exam domain each part answers;
- run it end to end, from a lesson to an approved question.

**Quick check:** before you move on, take this lesson's quick check. It's a few short questions, and it takes a couple of minutes.

## The layer map

Open one file per layer, and say out loud which exam domain it answers.

| Layer | Open this | Module | Exam domain |
|---|---|---|---|
| The call | `config.toml` (`[models]`, `[route.*]`, the budgets) | 1 | Model Selection and Optimization; API mechanics |
| The prompt | `prompts/item_writer/` (the versions and their lock) | 2 | Prompt and Context Engineering |
| The contract | `quizforge/schema.py` | 3 | Tools and MCPs; API mechanics |
| The agents | `quizforge/team.py` | 4 | Agents and Workflows |
| The workbench | `CLAUDE.md`, and the plugin's `hooks.json` | 5 | Claude Code |
| The server | `mcp_server/server.py` | 6 | Tools and MCPs |
| The proof | `evals/gate.py` | 7 | Eval, Testing, and Debugging |
| The guards | `quizforge/guards.py` | 8 | Security and Safety |
| The release | `quizforge/release.py` | 9 | Applications and Integration |

For each layer, find the control: the code that enforces a rule, whatever the model decides. A schema, a hook, a gate or a guard is a control; a line in a prompt or in `CLAUDE.md` is an instruction.

## Before you start

- Your `quiz-forge` project at the end of Lab 9, on `main`, with a clean `git status`.
- Your API key in `.env`.
- Cost: one `qf write` for one lesson, on Haiku. On the recording machine, that was six requests for less than half a cent.

## The run (the lesson's order)

Run these in the project folder. On Windows, the venv's programs are in `.venv\Scripts\` instead of `.venv/bin/`.

1. `git log --oneline --first-parent | head -14`, then `git tag`. Check that every release has a tag.
2. `.venv/bin/python -m pytest -q`. Every test passes, and none calls the API.
3. `.venv/bin/qf write --lesson L3.2` (or any lesson in your library). The writer drafts and checks its quotes; the checker reviews in a clean context. Read the checker's verdict for each question.
4. `.venv/bin/qf review --lesson L3.2`. Read each question in full (the options, the answer, the reason, the quote, the verdict) before you approve it. Decline anything you wouldn't put in front of a learner.
5. `.venv/bin/qf export --format md`, then open the file in `runs/exports/`. Your approved question is there, under its exam domain.
6. `.venv/bin/python -m quizforge.release --check`. Expect exit 0. Read any "watch" line: it names a model with a retirement date, which is your next upgrade, through the evals first.
7. `.venv/bin/qf report --since 1d`. Check that each feature is inside its budget.
8. `git status`. Only data should have changed (`bank/`). If any code changed, find out why before you go on.

## What the lesson printed

These results are from the recording machine (2026-10-13). Model output varies, so your questions will differ.

- **The tests:** `350 passed`.
- **`qf write --lesson L3.2`:** two questions saved to pending; the checker found both quotes supported; 6 requests, $0.00421.
- **`qf review`:** one question approved.
- **`release --check`:** Quiz Forge 1.0.0, every model active, every prompt locked, exit 0, with one "watch" line for the batch route's Haiku 4.5 (it retires no sooner than October 15, 2026).
- **`qf report --since 1d`:** 519 requests, $1.9677, over the project's last day.
- **`git status`:** the new approved question, and `bank/approved/MANIFEST.sha256`, sealed again. No code changed.
