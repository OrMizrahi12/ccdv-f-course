# Lesson 5.5: Plugins

The repository marketplace and its two plugins from the lesson, exactly as they were committed on camera (commit `7320bf8`, "Package the Quiz Forge tools as plugins in a repo marketplace", tagged `qf-format--v1.0.0` and `quiz-forge-tools--v1.0.0`, on top of Lesson 5.4).

| File | What it is |
|---|---|
| `plugin/.claude-plugin/marketplace.json` | the repository's marketplace: `quiz-forge-tools` and `qf-format`, version 1.0.0 each |
| `plugin/quiz-forge-tools/` | the `write-exam-item` and `new-items` skills, the `item-reviewer` subagent, the guard and test-gate hooks; depends on `qf-format` `^1.0` |
| `plugin/qf-format/` | the formatter hook (PostToolUse `Edit\|Write`) |
| `.claude/settings.json` | the marketplace (`extraKnownMarketplaces`) and both plugins enabled for everyone who clones; the old `hooks` block is gone |
| `tests/test_hooks.py` | the hook tests, now pointing at the plugin's scripts |

The hooks find their scripts with `${CLAUDE_PLUGIN_ROOT}` and the project with `$CLAUDE_PROJECT_DIR`. On Windows, use `python` instead of `python3` in the hook commands.

```bash
claude plugin validate ./plugin        # ✔ Validation passed (also each plugin folder)
python -m pytest -q tests/test_hooks.py   # 5 passed on camera
```

No API spend: Claude Code runs on your subscription. The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
