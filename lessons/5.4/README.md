# Lesson 5.4: Hooks: deterministic actions

The three hooks, their settings, the ruff config and the hook tests from the lesson, exactly as they were committed on camera (commit `54011a6`, "Hooks: format on edit, guard bank/approved, tests on stop", on top of `756646b`, the one-time `ruff format .`, and `ec3c751` from Lesson 5.3).

| File | What it is |
|---|---|
| `ruff.toml` | the formatter's config: `line-length = 120` |
| `.claude/hooks/guard_approved.py` | PreToolUse (`Edit\|Write\|Bash`): blocks Edit and Write paths inside `bank/approved/`, and Bash commands that mention `bank/approved` or `quizforge review`; exit 2 with the reason on stderr |
| `.claude/hooks/format.py` | PostToolUse (`Edit\|Write`): runs `ruff format` on an edited `.py` file; always exit 0 |
| `.claude/hooks/tests_on_stop.py` | Stop: when a Python file changed, runs `pytest -q -x`; on a failure, the last 20 lines to stderr and exit 2; lets a second stop through (`stop_hook_active`) |
| `.claude/settings.json` | the Lesson 5.2 permissions plus the `hooks` block; the guard has `"onFailure": "block"` |
| `tests/test_hooks.py` | five tests that run the hook scripts with an event on stdin |

Test each hook with a pipe before any session trusts it. On Windows, use `python` instead of `python3` in the hook commands, and the `.venv\Scripts\` paths in `format.py` and `tests_on_stop.py`.

```bash
echo '{"tool_name":"Write","tool_input":{"file_path":"bank/approved/x.json"}}' | python3 .claude/hooks/guard_approved.py; echo $?   # the reason, then 2
python -m pytest -q tests/test_hooks.py                                                                                           # 5 passed on camera
```

No API spend: Claude Code runs on your subscription, and the hooks make no API call. The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
