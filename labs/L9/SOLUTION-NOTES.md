# Lab 9 solution notes

Read these after you've tried the lab. They explain the decisions in `solution/`, the mistakes the self-check catches, and what to compare with your own version.

## The decisions, and why

- **The weights are shared over the covered domains only.**
  - The course library teaches four of the exam's seven domains (59% of the weight). Giving the other three their share would mean writing items no lesson supports, and FR-4 says every item quotes its lesson.
  - So the plan shares `n` over the four covered domains in proportion to their weights, and prints the other 41% as a coverage gap.
  - The gap is a finding for the course team, not something to hide.
- **Largest remainder, so the total is exact.** Weights times `n` rarely give whole numbers. If you round each domain on its own, a plan of 20 can come out as 19 or 21.
  - The largest remainder method gives every domain its whole part first, then the leftover items to the largest fractions.
  - So the total is always `n`, and no domain is off by more than one.
  - The same function splits a domain's share over its lessons, so the earlier lessons get the extra item.
- **No randomness, so the same inputs give the same plan.** A reviewer can rerun it, and a test can pin it. Ties go in the blueprint's own order.
- **The blueprint is checked against the library.** A lesson id that isn't in the library's `MANIFEST.json` stops the plan, with its name in the message. A typo in a blueprint should fail loudly before anything is generated, not produce a plan with an empty lesson.
- **The domain comes from the blueprint, not from the item.**
  - Items don't store their domain. The export looks it up when it writes the file.
  - Re-mapping a lesson then relabels every one of its items at the next export, and an old item can't carry a stale domain.
  - Without a blueprint, the column stays empty, and the export still works.
- **1.0.0, because the export broke.**
  - The CSV has a new column, in the middle. An importer that reads columns by position would now put the scenario in the wrong field.
  - Under semantic versioning that's a MAJOR change, and Quiz Forge's own changelog says the export format is something others rely on.
  - The changelog entry tells an importer exactly what to do.
- **The commit belongs to the code; the settings belong to the deployment.**
  - `release.py` asked git for the commit in `ROOT`. That's the deployment folder under `QF_HOME`, and it isn't a repository.
  - The commit identifies the code, so git runs in `CODE`. The config hash, the model lock and the prompt versions are the deployment's own, so they're still read from `ROOT`.
  - The same split, code from `CODE` and settings from `ROOT`, is what Lesson 9.6 applied to `platforms.toml`.
- **The fresh-clone check follows the README, not the developer's memory.**
  - The check runs every step a new team would, every time, with no API key. In Lesson 9.6, a fresh clone done by hand found a README that skipped the dev extra; this script makes that check repeatable.
  - It checks the program's real messages. An empty bank's export prints `nothing approved yet`, and the script compares that exact text.
  - When a step fails, the fix goes into the code or the README. The script stays as it is.

## Mistakes the self-check catches

| Symptom | Cause | Test that fails |
|---|---|---|
| A 20-item plan has 21 items | each domain rounded on its own | `test_lab9_plan.py::test_largest_remainder_sums_to_n` |
| An uncovered domain gets items with no lesson | the weights shared over every domain | `test_lab9_plan.py::test_counts_follow_the_weights_over_the_covered_domains` |
| A misspelled lesson id plans silently | the blueprint isn't checked against `MANIFEST.json` | `test_lab9_plan.py::test_a_bad_blueprint_is_refused_with_its_reason` |
| A lesson's items counted twice | the same lesson in two domains | `test_lab9_plan.py::test_a_bad_blueprint_is_refused_with_its_reason` |
| The export fails in a fresh deployment | the domain lookup requires a blueprint | `test_lab9_export.py::test_without_a_blueprint_the_column_is_empty` |
| A pending item appears in the export | the export read `bank/**` | `test_lab9_export.py::test_only_approved_items_and_text_survives_a_round_trip` |
| `python -m quizforge.release` fails in a deployment | git ran in `ROOT` | `test_lab9_release.py::test_the_manifest_runs_from_a_deployment_folder` |
| The version is 1.0.0, but nobody is told what broke | no changelog entry, or one without the export change | `test_lab9_release.py::test_the_version_is_1_0_0_and_the_changelog_says_what_changed` |
| A new team can't start | the README skips a step | `fresh_clone_check.sh` |

## What to compare

- **Your domain mapping against the solution's.** Where you differ, can you point to the lesson objective that decided it? L1.4 (context limits and memory) is the debatable one: the solution puts it under Configuration and Knowledge Management.
- **Your changelog's breaking-change line.** Would an importer know what to do from it alone?
- **Your ADR's Security and procurement table.** Does every answer have a source and a date? Would the answers still be true if you moved to Claude Platform on AWS?
