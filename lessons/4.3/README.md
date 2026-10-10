# Lesson 4.3: Context engineering

The lesson's files, exactly as they were typed on camera (commit `Context engineering: one agent, four lessons, keep vs clear vs restart, measured`).

| File | What it is |
|---|---|
| `experiments/context_growth.py` | One agent (Lesson 4.2's loop with the writer's tools), the lessons named on the command line, three ways to handle its context. **Keep everything**: all the lessons in one conversation. **Clear old tool results**: the same conversation with context editing (beta header `context-management-2025-06-27`, edit `clear_tool_uses_20250919`: once the input passes 10,000 tokens, clear the oldest tool results, keep the two most recent tool uses, clear at least 3,000 tokens at a time). **Restart per lesson**: a fresh conversation for each lesson. Limits: 30 turns and 60 cents per task. No person here: every save that passes the checks goes through and is counted. For each way it prints the requests, input tokens, the peak input of one request, the cost, how many times Claude read a lesson, how many lessons were saved, and how the runs stopped |

Run it from your `quiz-forge` project (on top of Lesson 4.2), with the environment active:

```bash
python -m experiments.context_growth L1.1 L1.2 L1.3 L1.4   # three ways, a few minutes
```

37 Claude Haiku 4.5 requests on camera: about 35 cents. On camera: keep everything 3 requests, peak 18,705 input tokens, cut off, 0 of 4 saved ($0.0497); clear old tool results 17 requests, 14 lesson reads, cut off, 0 of 4 saved ($0.1868); restart per lesson 17 requests, peak 7,453, 4 of 4 saved ($0.1142). The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
