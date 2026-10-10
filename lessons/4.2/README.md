# Lesson 4.2: Building the agent loop

The lesson's files, exactly as they were typed on camera (commit `An agent loop with explicit stops: turns, a budget, a person before the save`).

| File | What it is |
|---|---|
| `quizforge/agent.py` | The loop. `Stop` is an exception a tool raises when its decision ends the task. `Limits(max_turns=8, max_usd=0.05)`: the most requests one task may send, and its budget. `Outcome(stopped, turns, text, calls, peak)`: how it stopped, after how many turns, Claude's last text, the tools it called, and the largest input of any one request. `run(meter, cfg, system, tools, act, ask, limits)`: before every request, `count_tokens` and the worst case (the counted input plus `max_tokens` of output, priced); if spent plus worst case is over the budget, it returns `budget` without sending. Then it sends through Lesson 4.1's meter, prints one line per turn, and returns `done` on `end_turn`, `refused` or `cut off`, the tool's reason when a tool raises `Stop`, or `turn limit`. Every `tool_use` gets its `tool_result` |
| `quizforge/writer_tools.py` | The writer's tools: `read_lesson` (from `quizforge/tools.py`), `check_quote` (from the sources module) and `save_questions`, whose description says what the code checks before it saves. `save_questions` has two checkpoints: the code checks (the problems go back to Claude to fix), then a person (`approve`); a no raises `Stop("declined")`. Only then does it write `runs/agent/<lesson>.json`. `act(cfg, approve)` returns the dispatcher the loop calls for each tool use: unknown names and failed checks go back as text with `is_error`; `Stop` passes through to the loop |
| `experiments/agent_stops.py` | The writer agent four times on one lesson, each run with its own limits: enough room with a person who approves, a 2-turn limit, a 2-cent budget, and a person who declines. `person(data)` prints the start of each question that passed the checks and asks `Save these questions? [y/n]`; only `y` approves. Each run gets a fresh meter and prints how it stopped, the turns, the requests, the cost, and Claude's last words |

Run it from your `quiz-forge` project (on top of Lesson 4.1), with the environment active:

```bash
python -m experiments.agent_stops L1.3   # answer y in the first run, n in the fourth
```

Four runs on lesson L1.3, 11 Claude Haiku 4.5 requests: about 7 cents. On camera: done after 4 requests ($0.0270, approved); the turn limit after 2 ($0.0097); the budget after 2 ($0.0098 spent, the next request could cost $0.0140, so it was never sent); declined after 3 ($0.0192, nothing saved). The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
