# Lesson 5.2: Durable project context

The context and settings files from the lesson, exactly as they were committed on camera (commit `b4d214d`, "Durable context: a lean CLAUDE.md, an item rule, shared settings", on top of `641db08` from Lesson 5.1), plus the two personal files the lesson keeps out of git.

| File | Shared? | What it is |
|---|---|---|
| `CLAUDE.md` | committed | the lean project file after `/init` and pruning: the rules, and the Commands section (20 lines) |
| `.claude/rules/items.md` | committed | the practice-question rules, with a `paths` field: they load only when Claude reads a bank, schema, validator or item-writer prompt file |
| `.claude/settings.json` | committed | `defaultMode: default` (manual), allow the test run and `quizforge bank`, deny reading `.env` and `.env.*` |
| `.gitignore` | committed | adds `CLAUDE.local.md` and `.claude/settings.local.json` |
| `CLAUDE.local.md` | personal, ignored | two personal lines for this project; write your own |
| `.claude/settings.local.json` | personal, ignored | `defaultMode: plan` and one more allow rule; the lesson removes the `defaultMode` line at the end |

Your `/init` run will write a different CLAUDE.md: it reads your code. Prune it with one question: would Claude get something wrong without this line? On Windows, write the allow rules with the interpreter you actually run (`.venv\Scripts\python.exe`).

```bash
wc -l CLAUDE.md               # 11 lines before, 33 after /init, 20 after pruning
git status --short -uall      # four files to share, none of the personal ones
```

No API spend: Claude Code runs on your subscription. The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
