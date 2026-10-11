# Lab 6 solution notes

Read these after you've tried the lab. They explain the decisions in `solution/`, the mistakes the self-check catches, and what to compare with your own version.

## The decisions, and why
- **The server is a thin adapter.** `search_lessons` and `check_quote` call the same functions Quiz Forge's own commands use (`search_library`, `sources.check_quote`). A fix to the quote check lands once and reaches `generate`, Claude Code and `qf ask` together.
- **A custom tool and an MCP tool, side by side, on purpose.** Inside `generate`, the quote check stays a custom tool in Quiz Forge's own loop: no subprocess, no protocol, full control. The MCP server exists for the hosts that aren't Quiz Forge's own loop. `docs/decisions.md` records both choices with their deciding constraint.
- **`build(http)` instead of two servers.** One function assembles the resource, the tools and the prompt; only the transport and the auth differ. Two copies of the server would drift the first time a tool changes. The `hashes` argument exists for the tests: they register a token in a temporary file, so no real token is ever needed to prove the door is locked.
- **stdio gets no token check, and that is not a gap.** Over stdio the server is a child process of the host that started it, talking over its pipes; nobody else can reach it. Over HTTP anyone who can reach the port can knock, so every request must carry a token the verifier knows.
- **The boundary is the server's.** The library root comes from the server's own configuration. Nothing a client sends, in a tool argument or as a root (roots are deprecated in the 2026-07-28 specification), moves the server outside it.
- **Errors that teach.** An unknown lesson id returns the list of known ids; an empty query and a short quote say what to send instead. Claude corrects itself on the next call instead of guessing.
- **`qf ask` reuses the agent loop instead of the Tool Runner.** The SDK's MCP helper `mcp_tool()` does the conversion (schema in, tool call out, an MCP error raised as a tool error), and the Tool Runner could run those tools as they are. But the Tool Runner stops only on an iteration count. Module 4's `agent.run` already stops on a turn limit and on a budget checked *before* each request (spent so far plus the next request's worst case), and the meter logs every request. So `ask` hands the helper's tools to the loop it already trusts. The loop is synchronous and the MCP client is async: the loop runs in a worker thread, and each tool call goes back to the event loop through the helper.
- **Limits in configuration, not in code.** `[ask] max_turns = 5` and `max_usd = 0.05`. The reference question took 3 requests and $0.0056 on Haiku 4.5, so the limits leave room for a harder question without letting a confused one run on.
- **No token in configuration.** The HTTP entry uses `headersHelper`, which runs a script at connect time. The dev token lives in a mode-600 file outside the repository, and the verifier stores only its hash.

## Mistakes the self-check catches
| Symptom | Cause | Test that fails |
|---|---|---|
| Claude Code shows the server as failed, or the Inspector reports a parse error | a `print()` in the server writes to stdout, which is the wire | `test_mcp_server.py::test_server_never_prints` |
| The server works from the repo but not from Claude Code | paths relative to the working directory, not to the repository root | `test_mcp_server.py::test_paths_from_repo_root` |
| Claude keeps retrying the same bad lesson id | the error says "unknown" without the known ids | `test_mcp_server.py::test_unknown_lesson_names_known_ids` |
| Claude calls `get_objectives` to look for a topic | the description doesn't say what the tool can't do | `test_mcp_server.py::test_tools_are_read_only_and_say_what_they_cant_do` |
| The MCP tool and Quiz Forge disagree on a quote | the server re-implemented the check | `test_mcp_server.py::test_tools_share_quizforge_functions` |
| A request with no token reaches the tools over HTTP | the verifier isn't attached to the HTTP build | `test_mcp_http.py::test_missing_token_refused` |
| Any token works over HTTP | the verifier accepts what it doesn't know | `test_mcp_http.py::test_wrong_token_refused` |
| `ask` never ends on a confusing question | no turn limit on the loop | `test_ask.py::test_stops_at_turn_limit` |
| `ask` spends past its cap | the budget is checked after the request, or not at all | `test_ask.py::test_stops_before_the_budget` |
| Claude never learns a tool call failed | a tool error is returned as a normal result | `test_ask.py::test_a_tool_error_goes_back_as_an_error` |
| A teammate's Claude Code can call any tool of the server | a wildcard (`mcp__quiz-library__*`) in the allow list | `test_mcp_config.py::test_only_the_read_tools_are_allowed` |

## What to compare
- Your tool descriptions against the solution's: does each one say what it can't do?
- The input tokens of one `qf ask` request (about 1,000 to 1,800 in the reference run) against the same question with the whole lesson pasted in: the tools bring only the sections the question needs.
- Your `.claude/settings.json` allow list: exactly the three read tools, nothing with a wildcard.
- Your `ask` stop line on a question the library can't answer: `done` with an honest "not found", not a guess.
