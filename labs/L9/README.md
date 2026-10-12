# Lab 9: ship Quiz Forge 1.0.0, driven by an exam blueprint

Claude Certified Developer – Foundations (CCDV-F) · Module 9: Shipping & Reuse · the module checkpoint

Module 9 treated Quiz Forge as a product:
- **9.1:** requirements with numbers, and an architecture that meets them.
- **9.2:** one feature through the whole life cycle, from an issue to a reviewed, merged pull request.
- **9.3:** a refactor behind characterization tests, and a staged migration plan.
- **9.4:** a platform switch in the client factory, a capability check, and a decision record for where it runs.
- **9.5:** a release manifest, a model lock, semantic versions, and a model upgrade with its evidence.
- **9.6:** a package another team can install with `qf init` and run beside yours, with its data and identity kept apart.

One requirement from 9.1 is still open. **FR-2:** "Read an exam blueprint with domain weights", accepted when "items per domain match the weights". This lab closes it and ships the result as 1.0.0:

- **The blueprint drives the plan:**
  - `blueprint.toml` holds each exam domain, its weight, and the lessons that teach it.
  - `qf plan --n 20` splits 20 items over the domains, by weight, then over each domain's lessons.
  - A domain the library doesn't teach is reported as a coverage gap. It's never quietly given items.
- **The export names the domain:** each CSV row carries its lesson's domain, so the practice-test platform can build domain-weighted tests. The column is new, so the export format changes. That's a breaking change, so the version goes to 1.0.0.
- **The release runs from a deployment:** `python -m quizforge.release` must work under `QF_HOME`, in a deployment folder that isn't a git checkout.
- **Proof for a new team:** a script clones the repository and follows the README. It installs, runs `qf init`, runs the offline commands and the tests, and needs no API key.

You pass the lab when:
- the self-check tests pass;
- `scripts/fresh_clone_check.sh` ends with `fresh clone: ok`;
- 1.0.0 is tagged, with its changelog entry and decision record complete.

## What you start from

`starter/` is the `quiz-forge` project as it stands after Lesson 9.6, without build output (`quiz_forge.egg-info/`).

If your own project is there too, keep working in it. If not:
1. Copy `starter/` to `~/projects/quiz-forge`.
2. Add your `.env`.
3. Install with `.venv/bin/python -m pip install -e ".[dev]"`.

`solution/` is a finished version to compare with when you're done; `SOLUTION-NOTES.md` explains its main decisions. `tests/` holds the self-check.

## What you build

```
quiz-forge/
  blueprint.toml                 the exam's domains: name, weight, the lessons that teach it (weights add up to 100)
  quizforge/
    plan.py                      load() checks the blueprint against the library; split() and plan() share out n items
    export.py                    + a domain column, from the deployment's blueprint.toml
    init.py                      + a one-domain blueprint.toml for a new deployment
    release.py                   git runs in the checkout (CODE), not in the deployment folder (ROOT)
    cli.py                       + qf plan --n <items>
    __init__.py                  __version__ = "1.0.0"
  CHANGELOG.md                   ## 1.0.0: what broke, what's new, what's fixed
  docs/adr/0001-where-it-runs.md + ## Security and procurement, ## Re-check
  README.md                      + qf plan and qf export in the quickstart; the fresh-clone check
  scripts/fresh_clone_check.sh   clone, venv, install, qf init, the offline commands, the tests
  .gitignore                     + *.egg-info/
  tests/                         copy labs/L9/tests here
```

## Steps

### 1. The blueprint (Lessons 9.1, 9.6)

**Write `blueprint.toml`** at the project root, with one `[[domain]]` table per exam domain:
- `name`;
- `weight`, in percent;
- `lessons`: the library's lesson ids that teach that domain.

Then:
- Take the domains and weights from the exam guide. The course library's lesson L0.1 lists all seven.
- Map each lesson from its objectives. A lesson can be in one domain at most.
- Leave a domain's `lessons` out when the library doesn't teach it.
- Lessons in no domain (the orientation lessons, for example) get no planned items.

**Write `quizforge/plan.py`:**
- `load(path, library)` returns the domains, and raises `BlueprintError` with the reason when:
  - the weights don't add up to 100 (within 0.5);
  - a lesson is in two domains;
  - a lesson isn't in the library's `MANIFEST.json`;
  - the file is missing.
- `split(weights, n)` uses the largest remainder method:
  1. Give every share its whole part.
  2. Give the leftover items to the largest fractions, with ties going in listed order.

  So the total is always exactly `n`.
- `plan(domains, n)` returns `(rows, gaps)`:
  - **rows:** `(domain, lesson, items)`. It splits `n` over the domains that have lessons, by weight, then splits each domain's share over its lessons.
  - **gaps:** the domains with no lessons, with their weights.
  - If `n` is smaller than the number of covered domains, it raises an error.
- `qf plan --n <items>` prints the table, a total, and the coverage gap. It makes no API call.

**In `qf init`:** write a `blueprint.toml` with one domain holding every lesson, at weight 100. The new team then splits it into their own exam's domains.

### 2. The domain in the export (Lessons 9.2, 9.6)

**Add `domain` to `export.CSV_COLUMNS`, right after `lesson`.** Then:
- Fill it from the deployment's `blueprint.toml`.
- Leave it empty for a lesson that's in no domain, or when there's no blueprint.
- In the Markdown review sheet, add a `Domain:` line under each lesson's heading.
- Update the existing test that pins the columns. It's supposed to fail first: that's the breaking change made visible.

### 3. The release, from a deployment (Lessons 9.5, 9.6)

**The bug:** under `QF_HOME`, `release.manifest()` runs `git rev-parse` with `cwd=ROOT`. That's the deployment folder, which isn't a git repository, so it fails.

**The fix:** the commit belongs to the code, so run git in `CODE`. The deployment's own `config.toml`, `models.lock.toml` and prompts still come from `ROOT`.

### 4. 1.0.0 (Lesson 9.5)

1. Set `__version__ = "1.0.0"`.
2. Add a `## 1.0.0 (<date>)` entry at the top of `CHANGELOG.md`:
   - **Breaking:** the export's new column, and what an importer must do about it.
   - **New:** the blueprint and `qf plan`, and `qf init`'s blueprint.
   - **Fixed:** the release under `QF_HOME`.
   - Name the models and prompt versions it ships with, or say they haven't changed since the last release.
3. Build output doesn't ship. If `quiz_forge.egg-info/` is in git (`pip install -e .` creates it, and a `git add -A` picks it up), add `*.egg-info/` to `.gitignore` and run `git rm -r --cached quiz_forge.egg-info`.
4. Run `python -m quizforge.release --check`. It must exit 0. (It reads the Models API; that costs nothing.) A model whose locked `retires_not_before` date is less than 60 days away shows `active, watch`. It isn't an error: the API still says it's active. It means you check the deprecations page, and plan the upgrade the way Lesson 9.5 did, with evals, before a notice forces it.

### 5. The decision record, complete (Lesson 9.4)

Add two sections to `docs/adr/0001-where-it-runs.md`:
- **`## Security and procurement`:** a table with one row per question a reviewer asks. Give each answer and its source, with the date you read the source. The questions:
  - who processes the data, and where inference runs;
  - what you send;
  - how access is controlled;
  - how it's billed, and what it costs at your volume.
- **`## Re-check`:** a date, and what to re-read on that date: the five platform pages, then `platforms.toml`, `python -m quizforge.platforms` and latency. Add the monthly deprecations check.

### 6. The fresh clone (Lesson 9.6)

Write `scripts/fresh_clone_check.sh`. It needs no API key. It runs these steps in order, printing each one, and exits 1 at the first failure:
1. Clone the repository into a temporary folder.
2. Make a new venv and run `pip install -e ".[dev]"`.
3. Run `qf --help`.
4. Run `qf init` on `examples/sample-library/v1`.
5. With `QF_HOME` set to the new deployment, run:
   - `qf plan --n 6`;
   - `qf bank`;
   - `qf export --format csv`, which should report that nothing is approved yet;
   - `python -m quizforge.platforms`.
6. Run the tests.

At the end it prints `fresh clone: ok`. When a step fails, fix the README, not the script: the README is what a new team reads.

Add `qf plan` and `qf export` to the README's quickstart, and mention the check.

### 7. Check your work

1. Copy `labs/L9/tests/` into your project's `tests/`, then run:
   ```bash
   .venv/bin/python -m pytest -q
   bash scripts/fresh_clone_check.sh
   ```
   Every test must pass, and the check must end with `fresh clone: ok`. Neither needs an API key.

   **What the tests check:**
   - **The plan:**
     - the counts follow the weights and add up to `n`;
     - the same inputs give the same plan;
     - each bad blueprint is refused with its reason;
     - the course blueprint matches the library;
     - `qf plan` prints its table and the coverage gap;
     - `qf init` writes a blueprint.
   - **The export:**
     - the domain comes from the blueprint, or is empty without one;
     - only approved items are exported;
     - quotes and newlines survive a round trip.
   - **The release:** the version is 1.0.0, its changelog entry exists, and the manifest runs from a folder that isn't a checkout.
   - **The documents:** the ADR's sections and re-check date, the README's quickstart, and the fresh-clone script.
2. Then the real thing, with your key in `.env`:
   ```bash
   .venv/bin/python -m quizforge plan --n 20
   .venv/bin/python -m quizforge.release --check
   .venv/bin/python -m quizforge write --lesson L3.1
   .venv/bin/python -m quizforge review --lesson L3.1
   .venv/bin/python -m quizforge export --format csv
   ```
   `write` drafts the questions with the writer agent, has a checker read them in a clean context, and puts them in `bank/pending/`. It runs on your pinned model and costs well under a cent. `review` shows each question with its quote and the checker's verdict: answer `y` only for questions you have checked yourself against the lesson. Then open `runs/exports/approved.csv` and find the domain column.
3. Ship it through your own flow:
   1. Work on a release branch.
   2. Open a pull request, and wait for the checks to go green.
   3. Get a review, then merge.
   4. Run `git tag v1.0.0` on the merge commit, then `git push --tags`.

## Rubric (what the tests can't see)

| Check | Pass when |
|---|---|
| The blueprint is the exam's | The domains and weights match the exam guide, with the page and the date you read it |
| The mapping is defensible | Each lesson's domain follows from its objectives; a lesson that fits no domain stays out |
| The gap is visible | The coverage gap `qf plan` prints is in your release notes, not hidden by re-weighting |
| The break is announced | The changelog tells an importer exactly what changed in the export and what to do |
| The README is enough | Someone new gets from `git clone` to a planned item using only the README |
| The ADR answers both reviewers | Every security and procurement question has an answer, a source and a date |

## What a passing run looks like

- `qf plan --n 20` on the course library:
  - 20 items over the four domains the library teaches;
  - one line saying that 41% of the blueprint isn't covered: Workflow Integration and Solution Design, Governance, Risk, and Responsible Use, and Troubleshooting and Optimization.
- `python -m quizforge.release --check`:
  - `quiz-forge 1.0.0 at <commit>`, and the config hash;
  - each feature's model, `active`, with the date it retires no earlier than;
  - each prompt with its locked hash;
  - exit code 0.
- One `write` for L3.1: 2 questions in `bank/pending/`, both supported by the checker, 6 requests, about $0.004.
- `runs/exports/approved.csv` opens in a spreadsheet with one item per row, and a `domain` column after `lesson` (L3.1's rows say `Output Evaluation and Validation`). The Markdown sheet has a `Domain:` line under each lesson.
- `pytest`: 350 passed. `fresh_clone_check.sh` prints each step and `fresh clone: ok`.

When this lab was built (2026-10-12), `--check` showed one `active, watch`: the batch route's `claude-haiku-4-5-20251001`. The Models API still reported it active, but the lock's `retires_not_before` date was 2026-10-15. That's the monthly check doing its job. The answer is a model upgrade with its evidence, the way Lesson 9.5 did it, not a change to the check.

## Going further (optional)

- **Write a blueprint for a different course,** with three domains, in a `qf init` deployment, and run `qf plan`. If anything had to be edited by hand outside `blueprint.toml`, it belongs in `qf init`.
- **Make `qf generate` follow the plan:** a `--plan` flag that writes each lesson's planned count, through every guard from Module 8. What should happen when a lesson can't produce its share?

## Sources

- **Exam domains and weights:** the Claude Certified Associate exam guide, summarized in the course library's lesson L0.1.
- **Semantic versioning:** https://semver.org
- **The decision record's sources** (as read for Lesson 9.4):
  - Claude Platform on AWS: https://platform.claude.com/docs/en/build-with-claude/claude-platform-on-aws
  - Amazon Bedrock: https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock
  - Google Cloud: https://platform.claude.com/docs/en/build-with-claude/claude-on-vertex-ai
  - Microsoft Foundry: https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry
  - Data residency: https://platform.claude.com/docs/en/manage-claude/data-residency
  - Pricing: https://platform.claude.com/docs/en/about-claude/pricing
- **Model deprecations, for the monthly check:** https://platform.claude.com/docs/en/about-claude/model-deprecations
