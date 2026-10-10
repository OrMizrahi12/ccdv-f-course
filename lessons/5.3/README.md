# Lesson 5.3: Skills, commands and subagents

The skill, the command, the subagent and the subagent's memory from the lesson, exactly as they were committed on camera (commit `ec3c751`, on top of `b4d214d` from Lesson 5.2).

| File | What it is |
|---|---|
| `.claude/skills/write-exam-item/SKILL.md` | the skill: a description (always in context), an argument, one allowed tool (the checker), then the standard, the steps and a reviewer checklist (loaded on use) |
| `.claude/skills/write-exam-item/example-item.json` | a supporting file: one question in the bank's shape, read only when a step points to it |
| `.claude/skills/write-exam-item/check_item.py` | runs Quiz Forge's own checks on a pending file: the schema, the validator, the quote's section, item memory; prints `ok` or one line per problem |
| `.claude/skills/new-items/SKILL.md` | the command `/new-items <lesson> <count>`: `disable-model-invocation: true`, so only a person starts it |
| `.claude/agents/item-reviewer.md` | the subagent: tools `Read, Glob, Grep`; model `claude-haiku-4-5-20251001`; `memory: project`; the skill preloaded |
| `.claude/agent-memory/item-reviewer/` | the reviewer's memory after the lesson: `MEMORY.md` (the index) and the two notes it wrote |

Your skill and reviewer will word things differently: the model writes them. On Windows, the checker's allowed-tool rule uses `.venv\Scripts\python.exe`.

```bash
python .claude/skills/write-exam-item/check_item.py bank/pending/L1.3.json   # ok: 2 question(s) pass the checks, on camera
```

In Claude Code: `/skills` lists `write-exam-item`, and `/new-items L1.3 2` writes two questions and has the reviewer check them. No API spend: Claude Code runs on your subscription. The student kit has the full steps for Windows, macOS and Linux, and what the lesson printed.
