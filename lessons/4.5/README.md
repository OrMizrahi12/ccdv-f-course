# Lesson 4.5: Subagents and supervisors

The lesson's files, exactly as they were typed on camera (commit `Subagents and a supervisor: clean briefs, parallel subagents; one agent vs a sequence vs a team, measured`, on top of Lesson 4.4).

| File | What it is |
|---|---|
| `quizforge/team.py` | A supervisor and its subagents: `DELEGATE`, the supervisor's only tool (a lesson id and a self-contained brief); `subagent()`, Lesson 4.2's loop started from the brief alone, returning a one-line report; `supervise()`, the supervisor's loop, which runs the delegate calls of one turn at once in a thread pool, each subagent on its own meter; the supervisor's prompt |
| `experiments/team_vs_one.py` | The same job three ways: one agent with all the lessons in one context, one clean-context agent per lesson in sequence, and the supervisor with parallel subagents. Requests, input tokens, cost, wall time and lessons saved for each, and the supervisor's own share |

Run it from your `quiz-forge` project, with the environment active:

```bash
python -m experiments.team_vs_one L1.1 L1.2 L1.3 L1.4   # one agent, a sequence, a supervisor; about two minutes
```

39 Claude Haiku 4.5 requests on camera: about 29 cents. On camera: one agent cut off, 0 of 4 saved ($0.0693); in sequence 4 of 4 in 63.4 s ($0.1013); the supervisor 4 of 4 in 23.1 s ($0.1164), its own share 2 requests and 2,259 input tokens. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
