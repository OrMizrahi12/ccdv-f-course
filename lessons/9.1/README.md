# Lesson 9.1: From business need to requirements

Claude Certified Developer – Foundations (CCDV-F) · Module 9: Shipping & Reuse · Lesson 1

This folder holds the files from the lesson, exactly as they were committed on the recording machine on 2026-10-12. It also has the steps to repeat the lesson in your own `quiz-forge` project, starting from the end of Lab 8. The commits:

- `76b6e00` "A customer's request, and the requirements it becomes"
- `8269b40` "Residency checked on two models, and the quarter computed from our own logs"
- `10987c7` "The architecture: each requirement on the part that meets it"

By the end of the lesson you can:

- turn a business need into functional and infrastructure requirements;
- capture latency, volume, cost, compliance and residency;
- map requirements to an architecture.

**Quick check:** before you move on, take this lesson's quick check. It's a few short questions, and it takes a couple of minutes.

## What's in this folder

| File | What it is |
|---|---|
| `docs/request.md` | The customer's request. Northwind Learning is invented for this course. |
| `docs/requirements.md` | FR-1 to FR-7 (each with the phrase it came from and an acceptance test), NFR-1 to NFR-8 (each with a number and a check), and the open questions back to the customer. NFR-5 already holds the residency finding. |
| `experiments/residency.py` | One tiny request per model with `inference_geo="us"`. The model that answers also records the account's rate limits from the response headers, in `runs/limits.json`. |
| `experiments/capacity.py` | The quarter from your own logs: one lesson run on generate's route (the usage log), how many items the last eval gate judged good, and the load against the limits that `residency.py` recorded. |
| `docs/architecture.md` | The flow, and each part with the requirements it meets. |

The lesson changes no existing file.

## Before you start

- Your `quiz-forge` project at the end of Lab 8, with a clean `git status`.
- Your API key in `.env`.
- `runs/usage.jsonl` and `runs/last-gate.json` from your own earlier runs (the usage log your runs have written, and the eval gate from Lesson 7.4). `capacity.py` reads both.
- Cost: about $0.00001. `residency.py` sends two tiny requests: one is rejected with a 400, and one is answered.

## Steps (the lesson's order)

1. **The request, as requirements:** add `docs/request.md` and `docs/requirements.md`, then commit.
2. **Residency first:** add `experiments/residency.py`, then run `.venv/bin/python -m experiments.residency`.
3. **Volume and cost, from your logs:**
   1. Add `experiments/capacity.py`, then run `.venv/bin/python -m experiments.capacity`. It takes the approved items per quarter as an argument; the default is 2,000.
   2. Write the residency finding into NFR-5 (this folder's `docs/requirements.md` already has it), then commit.
4. **The map, and a review:**
   1. Add `docs/architecture.md`.
   2. Ask Claude Code to review it, in plan mode so it can't edit anything: `claude --permission-mode plan`. Then ask for every requirement that can't be tested as written, has no number, or isn't mapped to a part of the architecture.
   3. Exit Claude Code, then commit.

On Windows, the venv's Python is `.venv\Scripts\python`.

## What the lesson printed

These results are from the recording machine (2026-10-12). Your numbers come from your own logs and your own account, so they will differ.

**Residency:**

```
claude-haiku-4-5-20251001  400  'claude-haiku-4-5-20251001' does not support inference_geo.
claude-haiku-5-5           200  inference_geo=us  $0.000004  req_...
                           limits per minute: {'requests': 10000, 'input-tokens': 10000000, 'output-tokens': 2000000}
```

**Capacity:**

```
route: claude-haiku-5-5, 24 lesson runs in the usage log; gate 4/4 done, 8/8 items judged good
one lesson run: 3,353 input tokens, 1,102 output, $0.00092

per quarter                              value
approved items wanted                    2,000
lesson runs needed                       1,000
lesson runs per night                       15
input tokens per minute (night)            107
  limit (2026-10-12)                10,000,000
generate cost, realtime                  $0.92
  US-only (1.1x)                         $1.01
  US-only, Batch (50% off)               $0.50
```

8 of 8 items judged good is a small sample: a reviewer approves fewer, so 1,000 lesson runs is a floor.

**The review (Claude Code, plan mode):** it listed requirements that can't be tested as written (for example, FR-2 has no tolerance for the domain weights), numbers that are missing (the budget, the batch size per night), and weak mappings (FR-6 says one item, but the map says one lesson). It changed no file.
