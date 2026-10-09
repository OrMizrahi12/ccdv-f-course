# Lesson 3.1: Tool use from first principles

The lesson's files, exactly as they were typed on camera (commit `Tools by hand: one tool, descriptions measured, a loop with a dispatch table`, on top of Lab 2's Quiz Forge v0.2).

| File | What it is |
|---|---|
| `experiments/one_tool.py` | One tool, `read_lesson` (a description that says what it returns, when to use it and what it can't do; `lesson_id` an `enum` of the library's ids). Request 1 prints every content block, including the raw `tool_use` block; your code reads the lesson and sends it back as a `tool_result` with the matching `tool_use_id`; request 2 prints the question |
| `experiments/descriptions.py` | The same tool with a free-text `lesson_id`, a vague or a clear description, N runs: does Claude pass an id the library has? |
| `experiments/tool_loop.py` | Three tools, one `DISPATCH` table, a `run()` that returns failures as `is_error` results, a limit of eight turns, and `check_source`: an external call with an https-only host allowlist, a 10-second timeout and a User-Agent, all in code |

Run them from your `quiz-forge` project (Lab 2), with the environment active:

```bash
python -m experiments.one_tool
python -m experiments.descriptions vague 5
python -m experiments.descriptions clear 5
python -m experiments.tool_loop      # the answer goes to runs/tool_loop.md
```

Sixteen Claude Haiku 4.5 requests: about three cents. On camera: vague 0/5, clear 5/5; the loop made 4 requests (826, 1,121, 5,936 and 6,096 input tokens) and `check_source` returned HTTP 200 and the page title. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
