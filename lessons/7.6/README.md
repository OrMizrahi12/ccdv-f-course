# Lesson 7.6: Staying within budget

The code from the lesson, as it was committed on camera (`379706b`, `16892b3` and `5f2af32`).

| File | What it is |
|---|---|
| `quizforge/report.py` | Cost, requests, latency and cache reads per feature from `runs/usage.jsonl`, held against each feature's budget. Exit code 1 when a feature is over. |
| `docs/budgets.md` | The budget per feature, and the reasoning behind each number. |
| `experiments/route_sweep.py` | The same lessons on each candidate route (model, effort, max_tokens), each run graded, with cost per completed task and p95 latency. |
| `quizforge/budget.py` | One budget for several agents: a ledger that reserves each request's worst case before it is sent, then settles to the real cost. |
| `experiments/shared_budget.py` | Agents in parallel on one ledger: the run stops before it could cross its limit. |
| `tests/test_budget.py` | The ledger and the report, offline. |
| `lesson.diff` | The changes to files you already have: `[budget.*]` and `[route.generate]` in `config.toml`; the decision in `docs/decisions.md`; generate on its route in `generate.py` and `repair.py`; the ledger in `meter.py`; the route tests. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 7.5. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m quizforge.report; echo "exit code $?"
.venv/bin/python -m experiments.route_sweep
.venv/bin/python -m evals.run --cases regression.jsonl --gate --max-usd 0.10
.venv/bin/python -m experiments.shared_budget
.venv/bin/python -m pytest -q
```

Decide the route from your own sweep table before the gate run, and write the decision down. Cost: the sweep is about $0.20 (about 4 minutes), the gate about $0.03, and the shared run stops at $0.05 by construction; the report reads your log and is free. The student kit has the full steps in the lesson's order, and what the lesson printed.
