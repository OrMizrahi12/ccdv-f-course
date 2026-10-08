# Lesson 1.6: Choosing a model

The lesson's files, exactly as they were typed on camera (commit `List models, compare four, check an upgrade`).

| File | What it is |
|---|---|
| `experiments/models.py` | `client.models.list()`: every model your key can use, newest first, with its line and its input and output limits. Then `models.retrieve("claude-haiku-4-5")` shows which snapshot the alias points to, and a call through the alias shows the model that answered |
| `experiments/compare.py` | One quiz-question task sent to four models (Haiku 4.5, Haiku 5.5, Sonnet 5.5, Opus 5.5): seconds, input and output tokens, the cost from the pricing page's prices, and the question each one wrote |
| `experiments/upgrade.py` | Two Haiku 4.5 habits (`temperature`, an assistant prefill) and the migrated request, sent to Haiku 4.5, Haiku 5.5 and a retired model, with the status code and error message for each |

Run it from your `quiz-forge` project (Lessons 0.3 to 1.5), with the environment active:

```bash
python experiments/models.py    # your models, newest first; what the alias points to
python experiments/compare.py   # one task on four models: seconds, tokens, cost
python experiments/upgrade.py   # old habits on a new model: two 400s; a retired model: 404
```

The whole hands-on cost less than a cent. The 400 and 404 errors in `upgrade.py` aren't billed. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
