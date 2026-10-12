# Lesson 8.5: Secrets and identity

The code from the lesson, as it was committed on camera (`24cb820` and `28eaa84`).

| File | What it is |
|---|---|
| `quizforge/whoami.py` | Who is calling: the credential by name and source (never its value), and the workspace and request id from one free `count_tokens` call. |
| `quizforge/security/secret_scan.py` | Key-shaped strings in the staged changes, or in every commit with `--history`; masked hints, exit code 1 on a finding. |
| `.githooks/pre-commit` | The scan before every commit. |
| `tests/test_secret_scan.py` | A fake key built at runtime: found, with its place and a masked hint. |
| `docs/keys.md`, `docs/wif-migration.md` | The key runbook (one workspace and one kind of credential per environment), and the plan to move CI to workload identity federation. |
| `lesson.diff` | `review.yml` checks out the whole history and runs the scan after the tests. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 8.4. Copy the files in, then:

```bash
git apply lesson.diff
chmod +x .githooks/pre-commit && git config core.hooksPath .githooks
.venv/bin/python -m quizforge.whoami
.venv/bin/python -m quizforge.security.secret_scan --history
.venv/bin/python -m pytest -q
```

Cost: nothing; `count_tokens` is free. The student kit has the full steps in the lesson's order (including the blocked fake key), and what the lesson printed.
