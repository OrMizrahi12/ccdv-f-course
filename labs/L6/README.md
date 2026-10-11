# Lab 6: a course-library MCP server, used from Claude Code and from your own app

Claude Certified Developer – Foundations (CCDV-F) · Module 6: MCP & Agentic Customization · the module checkpoint

In Module 6 you met MCP from both sides: a public docs server in Claude Code and the protocol by hand (6.1); your own `quiz-library` server with lessons as resources, three read-only tools and a prompt, tested in memory, in the Inspector, in Claude Code and from an API app (6.2); the same server over Streamable HTTP, shared through `.mcp.json`, and the API's MCP connector (6.3); a measured choice between a built-in tool, a custom tool, a Skill and MCP (6.4); and an authenticated HTTP transport, a header helper instead of a stored token, and a security-review scope (6.5). In this lab the server becomes a finished part of Quiz Forge, and the API app from 6.2 becomes a command:

- `mcp_server/`: one server, two transports. `build(http=False)` for stdio (Claude Code and local apps); `build(http=True)` for Streamable HTTP with bearer-token checks. The same tools, resources and prompt either way, all read-only, all calling the same functions Quiz Forge itself uses.
- `qf ask "<question>"`: Quiz Forge's own MCP client. It starts the server over stdio, turns the server's tools into tools Claude can call with the SDK's MCP helper, and runs them in Module 4's agent loop, with its turn limit and budget. It answers with lesson ids.
- In Claude Code: the server is shared with the team in `.mcp.json`, and only its three read tools are allowed without asking.

You pass the lab when the self-check tests pass, the Inspector CLI lists the three tools, and one real `qf ask` answers with a lesson id.

## What you start from

`starter/` is the `quiz-forge` project exactly as it stands after Lesson 6.5 (commit "A security-review scope for connecting Claude to a legacy system"): Lab 5's setup, `mcp_server/server.py` with the `--http` switch and `mcp_server/auth.py`, `.mcp.json`, the read-tool allow rules in `.claude/settings.json`, `scripts/mcp_probe.sh`, `scripts/mint_dev_token.py`, `scripts/mcp_headers.sh`, `scripts/auth_probe.sh`, `docs/decisions.md` with the tool-choice section, `docs/security-review.md`, and the experiments `mcp_peek.py`, `library_api.py`, `mcp_connector.py` and `tool_choice.py`. Its 147 tests pass.

If your own project is there too, keep working in it; if not, copy `starter/` to `~/projects/quiz-forge`, add your `.env`, and install the pinned `requirements.txt`. Node.js is needed for the Inspector (`npx`), and Claude Code for step 2.

`solution/` is a finished version to compare with when you're done; `SOLUTION-NOTES.md` explains its main decisions. `tests/` holds the self-check: the project's own tests plus the lab's four new files.

## What you build

```
quiz-forge/
  mcp_server/
    server.py            build(http, hashes): resources, tools and the prompt; stdio by default, --http with auth
  quizforge/
    library_client.py    NEW: connect(): the server over stdio, as an MCP client
    ask.py               NEW: ask(meter, cfg, question, library): the server's tools in the agent loop
    config.py, cli.py    + the [ask] limits, + the ask command
  config.toml            + [ask] max_turns, max_usd
  CLAUDE.md              + ask in the Commands section (still 23 lines)
  tests/                 copied from labs/L6/tests
```

`.mcp.json`, `.claude/settings.json` and `mcp_server/auth.py` are already in the starter; step 2 checks them rather than rebuilds them.

The rules that carry over from the module: tools are model-controlled, resources app-controlled, prompts user-controlled; one implementation, many hosts; nothing on stdout but the protocol in a stdio server; tools say what they do and fail with an instructive error; a server enforces its own boundary (its library root, its scopes), whatever the client sends; stdio's boundary is the process that launched it, HTTP's is the token; no static token in any config file; least privilege in the host's permission layer too.

## Steps

### 1. One server, finished (Lessons 6.2, 6.3, 6.5)
`mcp_server/server.py`:
- **One build, two transports.** Move the resource, tools and prompt into `build(http: bool = False, hashes: Path = HASHES) -> MCPServer`. Only `build(http=True)` gets the token verifier (`HashedTokenVerifier(hashes)`) and `AuthSettings` (this server as the resource, `required_scopes=["library:read"]`, `validate_token_resource=True`). Keep `mcp = build()` at module level for tests and in-process clients. Under `__main__`: `--http` runs `build(http=True)` on `127.0.0.1:8765` with Streamable HTTP; otherwise `mcp.run()` serves stdio. The `hashes` argument lets a test register a token in a temporary file, never a real one.
- **Descriptions Claude can use.** Give each tool a `description` that says what it returns, when to use it and what it can't do (keep them as constants above `build`, so they read as one block):
  - `search_lessons`: three sections with their source; it can't match meaning, only words, and it searches only this library;
  - `get_objectives`: the objectives once you know the id; it can't search: find the lesson with `search_lessons` first;
  - `check_quote`: word for word in one lesson, or the closest sentence; it can't check a few words or anything outside the library.
- **Errors that teach.** An unknown lesson id in `get_objectives` names the ids the library has (`unknown lesson id 'L9.9': the library has L0.1, L0.2, ...`), like the resource already does. The empty query (`the query is empty: send a few key words, e.g. model tier latency`) and the short quote (`the quote is too short to check: send the full sentence the question quotes`) come from Quiz Forge's own functions: keep passing them on as tool errors.
- **One implementation.** `search_lessons` calls `quizforge.search.search_library`, `check_quote` calls `quizforge.sources.check_quote`. The server adds no logic of its own.
- **Paths from the repository root.** The library path comes from `config.toml` through `load_config()`, resolved against the repository, so the server works whatever folder the host starts it from.
- No `print()` anywhere in `mcp_server/`: on stdio, stdout is the wire.

### 2. Shared with the team, with least privilege (Lessons 6.3, 6.5)
Check, don't rebuild:
- `.mcp.json` (project scope, committed) starts `quiz-library` with `.venv/bin/python -m mcp_server.server`. A teammate approves it once, when they first start `claude` in the project.
- `.claude/settings.json` allows exactly `mcp__quiz-library__search_lessons`, `mcp__quiz-library__get_objectives` and `mcp__quiz-library__check_quote`, with no wildcard; any other MCP tool asks. The 6.1 deny rule for the public docs server's `submit_feedback` tool stays.
- No token in `.mcp.json` or the settings. The HTTP entry is personal (local scope) and uses `headersHelper` with `scripts/mcp_headers.sh`; rotating the token with `scripts/mint_dev_token.py` needs only a reconnect.

### 3. Quiz Forge as an MCP client: `qf ask` (Lessons 6.2, 4.x)
- `quizforge/library_client.py`: `connect()` returns an `mcp.Client` for `StdioServerParameters(command=sys.executable, args=["-m", "mcp_server.server"], cwd=<repo root>)`. The server runs as your own subprocess, with the same Python: no port, no token.
- `quizforge/ask.py`: `async def ask(meter, cfg, question, library, out=print)`:
  - list the server's tools and wrap each with the SDK's MCP helper `mcp_tool(tool, library.session)` (from `anthropic.lib.tools.mcp`); send `tool.to_dict()` for each as the request's tools;
  - run `agent.run(...)` from Module 4 in a worker thread (`anyio.to_thread.run_sync`), with a system prompt that asks for lesson ids and for "say so instead of guessing", and `agent.Limits(max_turns=cfg.ask_turns, max_usd=cfg.ask_usd)`. Its `act(call)` prints the tool call, runs `tools[call.name].call(call.input)` (the helper sends it back to the event loop and on to the server), and returns a tool error as `is_error`.
- `config.toml`: `[ask] max_turns = 5`, `max_usd = 0.05`; `Config` gains `ask_turns` and `ask_usd`.
- `cli.py`: `ask "<question>"` builds a `Meter` with `log=cfg.usage_log` and `label="ask"`, opens `connect()`, runs `ask`, and prints the answer, the stop (`done`, `turn limit` or `budget`) and the requests, input tokens and cost. Every request lands in `runs/usage.jsonl` with its request id.
- `CLAUDE.md`: add `ask "<question>"` (through the MCP server) to the Commands line.

### 4. Check your work
1. Copy `labs/L6/tests/` into your project as `tests/` and run:
   ```bash
   python -m pytest -q
   ```
   All 167 tests must pass. They need no API key. The lab's 20 tests are in four files:
   - `test_mcp_server.py`: every tool has a title, a read-only annotation and a description that says what it can't do; an unknown lesson id names the known ids; an empty query says what to send; the prompt stops at `bank/pending/` for `qf review`; the tools call Quiz Forge's own functions; nothing in `mcp_server/` prints; the server finds the library from any working directory.
   - `test_mcp_http.py`: the HTTP build answers 401 with `resource_metadata` without a token, 401 with a wrong one, and 200 to the registered one (a temporary hashes file); both builds are the same server.
   - `test_ask.py`: with the server in memory and a scripted stand-in for Claude, each tool call reaches the server and its result goes back; a tool error goes back as an error; `ask` stops at its turn limit and before its budget; `connect()` starts the real server over stdio and lists three tools; the limits come from `config.toml`.
   - `test_mcp_config.py`: `.mcp.json` shares the server, no token is in the shared configuration, and only the three read tools are allowed.

   On the starter, 12 of the 20 fail; the other 8 check what Module 6 already built.
2. Then the real thing, with the environment active:
   ```bash
   npx @modelcontextprotocol/inspector --cli .venv/bin/python -m mcp_server.server -- --method tools/list
   claude mcp get quiz-library
   python -m quizforge ask "Which lesson covers choosing a model tier, and what is one of its objectives?"
   python -m quizforge cost
   ```
   Note the `--` before `--method`: without it, the Inspector reads the server's `-m` as its own option. `ask` runs on Claude Haiku 4.5 and costs well under a cent.
3. Commit: `git add .` and `git commit -m "Quiz Forge v0.6: a finished course-library MCP server and qf ask, its own MCP client"`.

## Rubric (what the tests can't see)
| Check | Pass when |
|---|---|
| The right primitive | Lessons are resources (the app or person attaches them), lookups are tools (Claude calls them), the item workflow is a prompt (a person picks it) |
| Descriptions Claude can use | Each tool's description says what it returns, when to use it and what it can't do |
| One implementation | `git grep -n "search_library(\|find_quote(" mcp_server` shows the server only calling Quiz Forge's functions, and `def check_quote` exists once in `quizforge/` |
| Secrets stay out | No token in `.mcp.json`, the settings or the repository; the dev token file is mode 600 outside the repo |
| The choice is justified | `docs/decisions.md` says why the library is an MCP server and why the quote check inside `generate` is still a custom tool |

## What a passing run looks like (reference run)
- The Inspector CLI prints three tools: `search_lessons`, `get_objectives`, `check_quote`.
- `claude mcp get quiz-library` shows `Scope: Project config (shared via .mcp.json)`, `Type: stdio`, `Command: .venv/bin/python`, `Args: -m mcp_server.server`. In a fresh clone the status reads "Pending approval" until you start `claude` and approve the server.
- `qf ask` on Claude Haiku 4.5, in the reference run:
  ```
    turn 1: tool_use, ['search_lessons'], $0.0014 so far
    tool call  search_lessons {'query': 'choosing model tier'}
    turn 2: tool_use, ['get_objectives'], $0.0034 so far
    tool call  get_objectives {'lesson_id': 'L1.3'}
    turn 3: end_turn, [], $0.0056 so far
  ```
  then an answer naming lesson L1.3 and its objective "Tell Haiku, Sonnet and Opus apart: what each tier trades in capability, speed and cost", `stopped: done after 3 turn(s)` and `3 request(s), 4,469 input tokens, $0.00558`. Your wording and tool order may differ; the lesson id should not.

## Going further (optional)
- Point `qf ask` at the HTTP transport instead of stdio, with the token from your helper. What changes in `connect()`, and what stays the same in `ask()`?
- Add a fourth tool that writes something. Which annotation changes, and why does the allow list in `.claude/settings.json` stay as it is?
- Ask a question the library can't answer. Does `ask` say so, or guess? Which line of the system prompt decides that?

## Sources (as checked for Lessons 6.1 to 6.5)
- MCP introduction and architecture: https://modelcontextprotocol.io/docs/getting-started/intro, https://modelcontextprotocol.io/docs/learn/architecture
- MCP specification 2026-07-28 (tools, resources, prompts, transports, authorization): https://modelcontextprotocol.io/specification/2026-07-28
- MCP Inspector: https://modelcontextprotocol.io/docs/tools/inspector
- Connect Claude Code to tools via MCP: https://code.claude.com/docs/en/mcp
- MCP connector (API): https://platform.claude.com/docs/en/agents-and-tools/mcp-connector
- Tool use overview: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
