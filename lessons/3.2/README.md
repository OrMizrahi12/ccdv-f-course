# Lesson 3.2: Multi-turn and parallel tools

The lesson's files, exactly as they were typed on camera (commit `Tools in quizforge/tools.py, run at once; instructive errors and thinking blocks, measured`, on top of Lesson 3.1).

| File | What it is |
|---|---|
| `quizforge/tools.py` | Lesson 3.1's `experiments/tool_loop.py`, moved into the package with `git mv`: the three tools, `DISPATCH` and `run()`, plus `run_all(calls)`, which runs every `tool_use` block of one reply at once (a `ThreadPoolExecutor`) and returns the results in the order of the calls |
| `experiments/parallel.py` | The loop with `run_all`: each turn prints how many tool calls the reply holds, then how many results went back in one user message and how long the batch took. The report goes to `runs/sources.md` |
| `experiments/errors.py` | A failure sent back with `is_error: true`, as a generic message or as the library's own message (which names every valid id): how often does Claude recover? |
| `experiments/thinking_turns.py` | Extended thinking on Claude Haiku 4.5: the same tool turn sent back as received, without its thinking blocks, and with the thinking rebuilt without its signature |

Run them from your `quiz-forge` project, with the environment active (first `git mv experiments/tool_loop.py quizforge/tools.py`, then edit it to match):

```bash
python -m experiments.parallel
python -m experiments.errors generic 5
python -m experiments.errors instructive 5
python -m experiments.thinking_turns
```

About twenty-five Claude Haiku 4.5 requests: a few cents. On camera: one reply held 9 tool calls, and their 9 results went back in one user message in 1.0 s; generic errors 0/5, instructive errors 5/5; the thinking turn without its signature was rejected with a 400. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
