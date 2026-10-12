# Lesson 8.4: Hooks as guardrails

The code from the lesson, as it was committed on camera (`de9cd1f` and `0fa5505`).

| File | What it is |
|---|---|
| `quizforge/security/guard.py` | One pure policy for destructive actions: `decide(tool, args, root)` returns deny (with a reason), ask (with a reason), or allow, which means no opinion. A Bash line is split into simple commands, and each one is checked. |
| `plugin/quiz-forge-tools/scripts/guard.py` | The Claude Code PreToolUse hook: the event as JSON on stdin, a decision printed with its reason; nothing printed when the policy has no opinion. |
| `experiments/sdk_guard_demo.py` | The same policy as an Agent SDK hook callback: one `query()` on Haiku 4.5, Bash only, `dontAsk`, no project settings. |
| `tests/test_guard.py` | The policy as a table, and the hook script end to end. |
| `lesson.diff` | The changes to files you already have: the guard in the plugin's `hooks.json`, and two deny rules for force pushes in `.claude/settings.json`. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 8.3. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m pytest -q
echo '{"tool_name": "Bash", "tool_input": {"command": "rm -rf runs"}}' | python3 plugin/quiz-forge-tools/scripts/guard.py; echo "exit code $?"
mkdir -p runs/old-exports && touch runs/old-exports/export-1.json runs/old-exports/export-2.json
.venv/bin/python -m experiments.sdk_guard_demo
```

Cost: the SDK demo is one short query, about one cent; the tests and the hand-fed hook are free. The student kit has the full steps in the lesson's order (including Claude Code meeting the hook), and what the lesson printed.
