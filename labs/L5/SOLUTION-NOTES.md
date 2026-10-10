# Lab 5 solution notes

Read these after you've tried the lab. They explain the decisions in `solution/`, the mistakes the self-check catches, and what to compare with your own version.

## The decisions, and why
- **One verdict shape, generated in one direction.** `Verdict` in `reviewer/verdict.py` is the source; `verdict.schema.json` is generated from it and a test checks they match. In the starter the two were written separately and had already drifted: the file described each field and spelled "string or null" one way, the model another. Both still worked, which is how drift survives until the day the CI review and the SDK review disagree about a field.
- **`extra="forbid"` on the verdict.** The schema then says `additionalProperties: false`: a reviewer that invents a field fails validation instead of passing it through unread.
- **The plugin is validated in the `test` job.** A broken manifest or a missing hook script fails the build in seconds, before the `review` job starts spending on a model.
- **Hooks live only in the plugins.** Once the plugin carries the guard, the test gate and the formatter, a `hooks` block left in `.claude/settings.json` would run the same scripts twice, and the two copies would drift. `settings.json` only enables the plugin.
- **Two paths in every hook command.** `${CLAUDE_PLUGIN_ROOT}` finds the plugin's own scripts wherever the plugin is installed; `$CLAUDE_PROJECT_DIR` finds the project's `bank/` and tests. Mixing them up works in this repository and breaks in the next one.
- **The guard fails closed on Bash.** A Bash command that mentions `bank/approved` is blocked even when the script can't tell whether it writes (`cat bank/approved/x.json` is blocked too). So is Claude running `python -m quizforge review`: that command is the person's decision, not Claude's. A false block is cheap; a false pass is not.
- **The Stop hook checks `stop_hook_active`.** Without it, a test Claude can't fix keeps the session from ever ending. With it, the second stop goes through and the failure stays visible.
- **`dontAsk`, not `bypassPermissions`, for both automated reviewers.** In `dontAsk`, anything not pre-approved is denied; `disallowed_tools` removes Write, Edit, Bash and Agent outright. In `bypassPermissions` every tool would be approved.
- **A fresh `query()` per item.** Reviewing several items in one session makes each review more expensive (the context grows: 13,309 tokens at the first review, 40,232 at the fourth in Lesson 5.8) and lets one item's details reach another's verdict. Each item gets its own session id, and `--ask` resumes only that one.
- **Sonnet 5.5 for the reviewers that decide a build, a pinned Haiku 4.5 for the subagent.** In testing, Haiku 4.5 failed sound questions again and again as a headless reviewer; the in-session subagent's verdict is advice to the person, and its model id is pinned so the reviewer you calibrated is the one that runs.

## Mistakes the self-check catches
| Symptom | Cause | Test that fails |
|---|---|---|
| CI review rejects or misreads verdicts | the schema file was edited by hand and no longer matches `Verdict` | `test_verdict.py::test_schema_matches_model` |
| The SDK reviewer and CI disagree about a field | `sdk_review.py` still defines its own `Verdict` | `test_verdict.py::test_the_sdk_reviewer_uses_the_same_verdict` |
| A broken plugin reaches the review job | no `claude plugin validate` in the `test` job | `test_workflow.py::test_the_plugin_is_validated_before_any_review` |
| The guard "blocks" but the write still happens | the script exits 1 (a non-blocking error) instead of 2 | `test_hooks.py::test_guard_blocks_a_write_to_approved` |
| A session never ends when a test is broken | the Stop hook ignores `stop_hook_active` | `test_hooks.py::test_stop_hook_lets_a_second_stop_through` |
| The plugin is disabled after an update | the dependency has no range, or the formatter's version moved past it | `test_plugin.py::test_dependency_has_range` |
| A hook points at a script that isn't there | a path relative to the project instead of `${CLAUDE_PLUGIN_ROOT}` | `test_plugin.py::test_hook_scripts_exist` |
| Hooks run twice | a `hooks` block left in `.claude/settings.json` | `test_setup.py::test_settings_hold_no_hooks_the_plugins_carry_them` |
| The SDK reviewer edited a file | `bypassPermissions`, or Write/Edit/Bash not disallowed | `test_sdk_review.py::test_options_cannot_write` |
| A follow-up question answers about the wrong item | `--ask` resumes any session id it is given | `test_sessions.py::test_ask_refuses_another_items_session` |
| The key shows up in the repository | a key pasted into the workflow | `test_workflow.py::test_the_key_comes_from_a_secret` |

## What to compare
- Your `CLAUDE.md` against the solution's (23 lines): for each of your lines, would Claude get something wrong without it?
- Your guard's decisions on these commands: `mv bank/pending/a.json bank/approved/`, `cp -r bank/pending bank/approved`, `echo y | python -m quizforge review`. All three are blocked when Claude runs them; the third is yours to run.
- Your `verdict.schema.json` before and after step 2: `git diff reviewer/verdict.schema.json` shows exactly what had drifted.
- Your reviewer costs against their caps: about $0.13 for one SDK review (cap `[review] max_usd`) and $0.17 for one headless review (cap `--max-budget-usd 0.40`) in the reference run.
