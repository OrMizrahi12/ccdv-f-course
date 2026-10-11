# Lesson 6.3: Transports and deployment

The code from the lesson, as it was committed on camera (`dcb9e22`, `7c07433` and `931224c`).

| File | What it is |
|---|---|
| `mcp_server/server.py` | The library server from Lesson 6.2, plus: `check_quote` asks the person for a missing lesson (elicitation), and a `--http` switch that serves it over Streamable HTTP on 127.0.0.1:8765. |
| `tests/test_mcp.py` | The Lesson 6.2 tests, plus `test_no_lesson_asks_the_person` with an `elicitation_callback`. |
| `scripts/mcp_probe.sh` | Opens a conversation with curl two ways: the 2025-11-25 `initialize` handshake (a session id) and the 2026-07-28 `server/discover` (none). |
| `.mcp.json` | The server at project scope, as `claude mcp add --scope project` wrote it. |
| `experiments/mcp_connector.py` | One Messages API request with the MCP connector: a remote server, an `mcp_toolset` allow-list, and the API's reason when a request is refused. |

Start from your `quiz-forge` project at the Lesson 6.2 state. Copy the files in, then:

```bash
.venv/bin/python -m pytest -q tests/test_mcp.py
.venv/bin/python -m mcp_server.server --http > runs/http.log 2>&1 &
scripts/mcp_probe.sh
.venv/bin/python -m experiments.mcp_connector
```

Cost: the server, the tests and the probe make no model calls. `mcp_connector.py` cost $0.0291 on Haiku 4.5 when it was recorded. Claude Code runs on your own plan. The docs server the connector calls is a live public service, so its tools and answers can change. The student kit has the full steps for Windows, macOS and Linux, including the project and user scopes and the scope collision.
