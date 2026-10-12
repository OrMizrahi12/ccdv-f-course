# Lesson 9.2: The systems life cycle

Claude Certified Developer – Foundations (CCDV-F) · Module 9: Shipping & Reuse · Lesson 2

This folder holds the files from the lesson, exactly as they were merged on the recording machine on 2026-10-12. It also has the steps to ship the same feature in your own `quiz-forge` project, starting from the end of Lesson 9.1. The feature is FR-7 from Lesson 9.1's requirements: export the approved questions.

On `main`, the whole feature is one squashed commit, `815db83` "feat/export (#12)". On the branch it was four commits: the export with its tests, the `qf export` command, the review change (rows sorted by lesson and question), and the runbook.

By the end of the lesson you can:

- compare waterfall, iterative and DevOps life cycles;
- ship through branches, pull requests and reviews;
- fit Claude into each phase, with people deciding.

**Quick check:** before you move on, take this lesson's quick check. It's a few short questions, and it takes a couple of minutes.

## What's in this folder

| File | What it is |
|---|---|
| `quizforge/export.py` | `qf export`: reads `bank/approved/` only if it still matches its seal (Lesson 8.2), and writes `runs/exports/approved.csv` (one row per question; the columns are in `CSV_COLUMNS` at the top) or `approved.md` (a review sheet, grouped by lesson). Rows are sorted by lesson, then question. |
| `tests/test_export.py` | 13 tests, no network: commas, quotes and newlines stay in one cell, the columns drive the header and the rows, the seal check, an empty bank, the sort order, and the command. |
| `docs/runbook.md` | Operating the feature: how to run and check the export, what to check every day, and who decides. |
| `lesson.diff` | The change to an existing file: `quizforge/cli.py` gets the `export` command. Apply it with `git apply lesson.diff`, or make the same edits by hand. |

## Before you start

- Your `quiz-forge` project at the end of Lesson 9.1, on `main`, with a clean `git status`, pushed to your GitHub repository.
- The GitHub CLI (`gh`), signed in to your account.
- Your CI workflows from Module 7 (the eval gate) and Lesson 5.6 (the question review). The pull request runs them.
- Claude Code, signed in.
- Cost: the export makes no API call. Claude Code runs on your Claude plan or API key. Your CI's eval gate costs what it cost in Lesson 7.4.

## Steps (the lesson's order)

1. **The platform, the issue, the branch:**
   1. Check whether `main` can be protected: `gh api repos/{owner}/{repo}/branches/main/protection`. On GitHub Free, a private repository answers with a 403: protected branches need a public repository or a paid plan. If yours can be protected, require the checks and a review before merge.
   2. Open the issue: `gh issue create --title "Export approved items (FR-7)" --body "..."`, with its acceptance test (one row per approved question; tests cover commas, quotes and newlines).
   3. Create the branch: `git switch -c feat/export`.
2. **Plan, then build:**
   1. Start Claude Code in plan mode: `claude --permission-mode plan`. Ask for a plan for your issue: an export module that reads the approved bank, a CSV or a Markdown review sheet, a `qf export --format csv|md` command, tests first, no network, no commit.
   2. Read the plan. Send it back with your changes before you accept it (the lesson asked for the CSV columns in a constant, and a test for a stem with a comma, a quote and a newline).
   3. Accept it with "Yes, manually approve edits", and read each edit before you approve it.
   4. Exit Claude Code.
3. **Commit, and open a pull request:**
   1. Read what changed: `git status --short && git --no-pager diff --stat`.
   2. Commit in small steps: the export with its tests, then the command.
   3. Push, and open the pull request: `git push -u origin feat/export`, then `gh pr create --fill --body "Closes #<your issue>"`.
   4. Watch the checks: `gh pr checks --watch --interval 10`.
4. **Review, and merge:**
   1. A person reviews the pull request. The lesson's review asked for the rows sorted by lesson, then by question, with a test for the order.
   2. Have Claude Code address the comment: "Address the review comment on PR #<n> (gh pr view <n> --comments). Change only what it asks, run the tests, and don't commit." Approve each edit, then commit.
   3. Add `docs/runbook.md`, commit, and push. The checks run again.
   4. When the checks are green, a person merges: `gh pr merge --squash --delete-branch`. Then `git switch main && git pull`. The issue closes with the merge.
5. **Operate, and maintain:**
   1. Run the export: `.venv/bin/python -m quizforge export --format csv`.
   2. Run the daily check: `.venv/bin/python -m quizforge report --since 1d`.
   3. Open an issue to watch your pinned models' deprecation dates (NFR-8), and leave it open for Lesson 9.5.

On Windows, the venv's Python is `.venv\Scripts\python`.

## What the lesson printed

These results are from the recording machine (2026-10-12). Your issue and pull request numbers, check times and bank will differ.

**Branch protection (a private repository on GitHub Free):**

```
gh: Upgrade to GitHub Pro or make this repository public to enable this feature. (HTTP 403)
```

**The checks on the pull request:**

```
All checks were successful
0 cancelled, 0 failing, 4 successful, 0 skipped, and 0 pending checks

   NAME                                 ELAPSED
✓  Eval gate/gate (pull_request)        1m9s
✓  Review questions/review (pull_request)  20s
✓  Review questions/test (pull_request)    37s
✓  Eval gate/unit (pull_request)        28s
```

**The review change (Claude Code's note):** the sort is a plain string comparison, so a lesson like `L1.10` comes before `L1.3`. If your lesson ids go past 9, ask for a numeric sort.

**After the merge:**

```
* 815db83 (HEAD -> main, origin/main) feat/export (#12)
* 10987c7 The architecture: each requirement on the part that meets it
* 8269b40 Residency checked on two models, and the quarter computed from our own logs
* 76b6e00 A customer's request, and the requirements it becomes
CLOSED
```

**The export:**

```
2 question(s) -> .../quiz-forge/runs/exports/approved.csv
```
