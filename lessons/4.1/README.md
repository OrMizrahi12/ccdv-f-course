# Lesson 4.1: Workflow or agent?

The lesson's files, exactly as they were typed on camera (commit `Workflow or agent: the same job both ways, measured, and the decision recorded`, on top of Lab 3's Quiz Forge v0.3).

| File | What it is |
|---|---|
| `quizforge/meter.py` | `Meter(client, prices, model)` stands in for the client: code that calls `client.messages.create` gets the meter instead, unchanged (`meter.messages.create`). It counts the requests and the input and output tokens, prices each request from its usage, and times the job. `count_tokens` passes straight through: it's free, and not counted |
| `experiments/workflow_vs_agent.py` | The same job two ways, measured: two checked questions for one lesson. `python -m experiments.workflow_vs_agent <lesson> <runs>`. The workflow is `generate_one`, unchanged: our code sets the path, and repairs only when the checks fail. The agent is Claude with `read_lesson`, `check_quote` and `save_questions` (the save tool runs the same checks and refuses with `is_error` and the problems), the same locked prompt, and at most 8 turns. Each run prints its requests, input tokens, cost, time, whether it passed, and its path (the workflow's attempts, or the agent's tool calls); then the mean per run for each way, also written to `runs/workflow_vs_agent.json`. The agent's saved questions go to `runs/agent/` |
| `docs/decisions.md` | The decision record: the job, the four checks (complexity, value, viability, cost of error), the measured means, and the decision: `generate` stays a workflow |

Run it from your `quiz-forge` project, with the environment active (no new packages), then write your own numbers into `docs/decisions.md`:

```bash
python -m experiments.workflow_vs_agent L1.3 3   # three runs each way
```

About 16 Claude Haiku 4.5 requests: about 11 cents. On camera, three runs each on L1.3: the workflow passed all three with 1.3 requests and $0.0105 a run (one run needed a repair); the agent passed all three with 4 requests and $0.0261 a run. The decision, in `docs/decisions.md`: `generate` stays a workflow. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
