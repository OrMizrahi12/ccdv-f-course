# Lesson 1.7: Reasoning controls

The lesson's files, exactly as they were typed on camera (commit `Measure thinking and effort, map the older controls`).

| File | What it is |
|---|---|
| `experiments/thinking.py` | Counts the right answer to a hard puzzle by brute force, then sends an easy task and the puzzle to Claude Haiku 5.5 with adaptive thinking at low and high effort, and the puzzle once more with the thinking display omitted. Each line prints seconds, output tokens, thinking tokens (from `usage`), and how many characters of thinking text came back |
| `experiments/budget.py` | The same puzzle three ways (`budget_tokens: 2000`, `thinking: adaptive`, `effort: low`) on Haiku 4.5 and Haiku 5.5, with the error message for each request a model rejects |

Run it from your `quiz-forge` project (Lessons 0.3 to 1.6), with the environment active:

```bash
python experiments/thinking.py   # five calls: easy task and puzzle, low and high effort
python experiments/budget.py     # budget_tokens, adaptive, effort on two models (the Haiku 4.5 call takes ~40 s)
```

The whole hands-on cost about two cents, most of it the Haiku 4.5 budget call's thinking tokens. The 400 errors aren't billed. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
