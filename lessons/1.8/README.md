# Lesson 1.8: Tokens and money

The lesson's files, exactly as they were typed on camera (commit `Log usage, price the finished task, compare the levers`).

| File | What it is |
|---|---|
| `experiments/usage.py` | `PRICES` from the pricing page, the cache multipliers, a one-cent `LIMIT`, `cost()` from a response's `usage`, and `call()`: count the tokens first (`count_tokens`), refuse any request that could cost more than the limit, send it, and append one JSON line per request to `runs/usage.jsonl` |
| `experiments/tasks.py` | A real unit of work: five checked quiz questions about a 4,400-token lesson file. Each task writes a question, then checks it (PASS or FAIL), both through `call()`. At the end: the cost per request and the cost per completed task |
| `experiments/levers.py` | One request priced six ways: the baseline, a cache write, a cache read, a shorter output, Claude Haiku 5.5, and Haiku 5.5 at low effort |

Run it from your `quiz-forge` project (Lessons 0.3 to 1.7), with the environment active:

```bash
python experiments/tasks.py    # five checked questions: cost per request, cost per completed task
python experiments/levers.py   # one request priced six ways
```

The whole hands-on cost about seven cents: sixteen Claude Haiku 4.5 and Haiku 5.5 requests, each with a 4,400-token lesson. Token counting is free. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
