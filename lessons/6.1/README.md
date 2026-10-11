# Lesson 6.1: MCP from zero

The script from the lesson, as it was committed on camera (`e892efa`).

| File | What it is |
|---|---|
| `experiments/mcp_peek.py` | MCP by hand: connect to a server, print its name, protocol revision and capabilities, list its tools and resources, call its search tool. No model involved. |

Start from your `quiz-forge` project at the Lab 5 state, with the MCP Python SDK installed (`.venv/bin/python -m pip install mcp`; the lesson used 2.3.0). Add the public Claude Code docs server to Claude Code, then look at it by hand:

```bash
claude mcp add --transport http claude-code-docs https://code.claude.com/docs/mcp
.venv/bin/python -m experiments.mcp_peek
.venv/bin/python -m experiments.mcp_peek https://platform.claude.com/docs/mcp
```

To keep Claude from calling the server's feedback tool, add `"mcp__claude-code-docs__submit_feedback"` to `permissions.deny` in `.claude/settings.json`.

Cost: nothing. The script makes no model calls, and both docs servers are public. They are live servers, so their tools and protocol revisions can change. The student kit has the full steps for Windows, macOS and Linux.
