# Lesson 6.2: Build an MCP server in Python

The code from the lesson, as it was committed on camera (`ce81a40` and `98673c1`).

| File | What it is |
|---|---|
| `mcp_server/server.py` | The course-library MCP server: the lessons as a `lesson://{lesson_id}` resource; `search_lessons`, `get_objectives` and `check_quote` as read-only tools that raise `ToolError` on bad input; a `write-items` prompt. It runs over stdio, so it never prints. |
| `mcp_server/__init__.py` | Empty: makes `mcp_server` a package, so `python -m mcp_server.server` works. |
| `tests/test_mcp.py` | Five tests that talk to the server in memory, with no subprocess and no model. |
| `experiments/library_api.py` | The same server inside an API app: our own MCP client starts it over stdio, the SDK's helpers turn its tools into Tool Runner tools, and the lesson resource is attached and counted. |

Start from your `quiz-forge` project at the Lesson 6.1 state, with `mcp==2.3.0` and `anthropic==1.11.0` installed. Copy the files in, then:

```bash
.venv/bin/python -m pytest -q tests/test_mcp.py
npx @modelcontextprotocol/inspector@2.10.1 --tui -- .venv/bin/python -m mcp_server.server
claude mcp add --transport stdio quiz-library -- "$PWD/.venv/bin/python" -m mcp_server.server
.venv/bin/python -m experiments.library_api
```

Cost: the tests, the Inspector and the server make no model calls. `library_api.py` made 3 requests on Haiku 4.5 for $0.0050 when it was recorded; token counting is free. Claude Code runs on your own plan. The student kit has the full steps for Windows, macOS and Linux.
