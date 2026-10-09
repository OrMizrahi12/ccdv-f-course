# Lesson 3.3: Tool Runner and server tools

The lesson's files, exactly as they were typed on camera (commit `Tool Runner, an approval step, web fetch and tool search, measured`, on top of Lesson 3.2).

| File | What it is |
|---|---|
| `experiments/runner.py` | Lesson 3.2's source check with the SDK running the loop: the tools as plain functions under `@beta_tool`, `client.beta.messages.tool_runner(...)` with `max_iterations=8`, failures raised as `ToolError`. The report goes to `runs/runner.md` |
| `experiments/approval.py` | One tool with a side effect (`save_question`), behind an `Approve? [y/n]` step in our own loop; a declined call goes back as an `is_error` result |
| `experiments/server_tools.py` | Web fetch, a server tool, limited to `platform.claude.com`: one allowed page, one page off the list, one page asked for by description only |
| `experiments/tool_search.py` | Our three tools plus 36 stand-ins, sent once all loaded and once with the 36 deferred behind the tool search tool: the input tokens of each |

Run them from your `quiz-forge` project, with the environment active (`anthropic` 1.11.0 or later):

```bash
python -m experiments.runner
python -m experiments.approval
python -m experiments.server_tools
python -m experiments.tool_search
```

About a dozen Claude Haiku 4.5 requests: a few cents. On camera: the runner ran 9 calls one after another in 1.7 s; the approved question was saved; web fetch returned the docs page (8,028 input tokens), `url_not_allowed` and `url_not_in_prior_context`; 39 tools loaded cost 2,936 input tokens, 3 loaded with 36 deferred 1,019. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
