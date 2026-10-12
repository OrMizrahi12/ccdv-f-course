# Lesson 9.6: Accelerators and coordinated deployments

Claude Certified Developer – Foundations (CCDV-F) · Module 9: Shipping & Reuse · Lesson 6

This folder holds the files from the lesson, exactly as they were on `main` on the recording machine on 2026-10-12, after the pull request was merged. It also has the steps to repeat the lesson in your own `quiz-forge` project, starting from the end of Lesson 9.5. The commits:

- `cb67f61` "Quiz Forge as a template: a package, QF_HOME, and qf init for another library"
- `eb1c9ed` "Deployment boundaries: the library server told its deployment, a key per deployment, a fleet view without content"
- `0508747` "README and CONTRIBUTING"
- `9fc3593` "README: install the dev extra, or the tests can't run (found by a fresh clone)"
- `dc4fb0c` "An upstream issue draft: a directory marketplace runs another checkout's hooks"
- `7c1f988`: the merge of pull request 16 into `main`

By the end of the lesson you can:

- package an agent template, MCP server and evals;
- contribute back through documented channels;
- coordinate deployments across data and identity boundaries.

**Quick check:** before you move on, take this lesson's quick check. It's a few short questions, and it takes a couple of minutes.

## What's in this folder

| File | What it is |
|---|---|
| `pyproject.toml` | Quiz Forge as a package: one install, the same pinned dependencies as `requirements.txt`, and a `qf` command. |
| `quizforge/init.py` | `qf init`: a deployment folder for another course library, with its own copy of the library, settings, prices, model lock, prompts, eval cases, bank and logs. The writer's prompt gets the next version, with the new audience, locked. The template's own files are never edited. |
| `quizforge/fleet.py` | Several deployments, one view. Each deployment publishes `runs/status.json` (counts, cost, models, release), and the coordinator reads only those files. |
| `examples/sample-library/v1/` | The second team's library: three short lessons about making bread, with their `MANIFEST.json`. |
| `tests/test_init.py` | A deployment gets its own files; the audience gets a new, locked prompt version and the template stays; `init` refuses a folder in use. |
| `tests/test_boundaries.py` | What a deployment can reach: its own settings, library, bank and log; a lesson outside its library is refused; the library server is told which deployment it serves, and gets no key; the fleet view holds numbers and ids only. |
| `docs/deployments.md` | Each boundary, how it holds and the test that checks it. Then the identity for real use: a workspace, a credential and a spend limit per deployment. |
| `README.md` | Install, start a deployment for your own library, and check it. It's the version after the fresh clone's fix. |
| `CONTRIBUTING.md` | Quiz Forge's own channel: an issue first, a branch, a pull request with passing checks, evidence for model or prompt changes, and private reports for security problems. |
| `docs/upstream/claude-code-marketplace-path.md` | An issue draft for Claude Code, written the way a good report is. It's a draft, and it was never filed. |
| `lesson.diff` | The changes to files that already existed: `config.py` (`CODE` and `ROOT`), `platforms.py` (the table read from `CODE`), `cli.py` (the `init` command), `library_client.py` (the server is told `QF_HOME`, and gets no key) and `client.py` (the deployment's own `.env`). Apply it with `git apply lesson.diff`, or type the lines yourself. |

## Before you start

- Your `quiz-forge` project at the end of Lesson 9.5, on `main`, with a clean `git status`, pushed to your own GitHub repository with its CI.
- Your API key in `.env`.
- Cost: one `generate` and one `qf ask` in the bakery deployment, plus one `generate` in the fresh clone. On the recording machine, that was less than a tenth of a cent each. CI's eval gate runs on the pull request too.

## Steps (the lesson's order)

1. **A template, not a copy:**
   1. Make a branch: `git switch -c template/1.0`.
   2. Search for everything that names your course: `grep -rln "Certified Associate" quizforge mcp_server evals prompts config.toml`. Code that reads its paths from config shouldn't match.
   3. Add `pyproject.toml`.
   4. In `quizforge/config.py`, add `CODE` (this checkout) and `ROOT` (`QF_HOME`, or the checkout when it's not set).
   5. In `quizforge/platforms.py`, read `platforms.toml` from `CODE`.
   6. Add `quizforge/init.py`, and the `init` command in `quizforge/cli.py`.
   7. Add `examples/sample-library/v1/`.
   8. Install: `.venv/bin/python -m pip install -e .`.
   9. Run `.venv/bin/qf init ~/qf-deploy/bakery --library examples/sample-library/v1 --name bakery --audience "home bakers learning to make bread"`.
   10. Run `QF_HOME=~/qf-deploy/bakery .venv/bin/qf generate --lesson B1.1`, then `qf show`. Read every question before you approve it.
   11. Add `tests/test_init.py`, run the tests, and commit.
2. **A boundary per deployment:**
   1. Add `tests/test_boundaries.py`, and run it. Two tests fail: the library server, and the fleet view you haven't written yet.
   2. In `quizforge/library_client.py`, pass `QF_HOME` to the server. In `quizforge/client.py`, load the deployment's own `.env`. Run the tests again: one is left.
   3. Copy your key into `~/qf-deploy/bakery/.env`, then ask the bakery's agent a question: `QF_HOME=~/qf-deploy/bakery .venv/bin/qf ask "How long should a loaf cool before slicing?"`.
   4. Add `quizforge/fleet.py`. Publish each deployment's status, then show both: `python -m quizforge.fleet show . ~/qf-deploy/bakery`.
   5. Add `docs/deployments.md`, run the tests, and commit.
3. **A fresh clone, and giving back:**
   1. Add `README.md` and `CONTRIBUTING.md`, then commit.
   2. Clone your project into a new folder, and follow only the README: a new venv, the install, `qf init`, a key file, one `generate`, and the tests.
   3. When a step fails, fix the README, not your memory. Commit, then clone again.
   4. If you've found a problem in a tool you use, write the report the way its project asks: the version, a minimal reproduction, what you expected and what happened.
   5. Push, open one pull request, wait for the checks, then merge with a merge commit.

On Windows, the venv's Python is `.venv\Scripts\python`, and you set `QF_HOME` with `$env:QF_HOME = "..."` in PowerShell.

**One thing to fix in your own project:** `pip install -e .` creates a `quiz_forge.egg-info/` folder, and on the recording machine, `git add -A` committed it with the template. It's build output: add `*.egg-info/` to `.gitignore`, and remove it from git with `git rm -r --cached quiz_forge.egg-info`. Lab 9 does this.

## What the lesson printed

These results are from the recording machine (2026-10-12). Model output varies, so your questions and answers will differ.

**What names our course:** `evals/golden.jsonl`, the writer's prompts (`system.md`, `v2.md`, `v3.md`, `v4.md`) and `config.toml`. No code file matched.

**`qf init`:** `library ... (3 lessons)`, `prompt v5: v4 for home bakers learning to make bread, locked`, `eval cases: B1.1, B1.2, B1.3`.

**The bakery's first questions:** `B1.1: passed after 1 attempt(s), $0.00043`. Question 1 had a flawed key that every check passed. The key, A, gave the right number (16 grams) with the wrong reason ("salt is measured by the cup"). That's why a person approves every item.

**The boundary tests:** first `2 failed, 2 passed`; after the two fixes `1 failed, 3 passed`; with `fleet.py`, `330 passed, 4 deselected`.

**The fleet view:**

```
deployment    release  prompt  lessons  pending  approved  requests     usd  models
quiz-forge     0.10.0      v4       14        2         1      1052  4.3844  claude-haiku-5-5, claude-haiku-5-5
bakery         0.10.0      v5        3        0         0         3  0.0008  claude-haiku-5-5, claude-haiku-5-5
```

**The fresh clone:** `generate` passed ($0.00039), then the tests failed with `No module named pytest`: the README's install line skipped the dev extra. After the fix, the clone again: `330 passed, 4 deselected`.

**The pull request:** all 4 checks successful, merged with a merge commit.
