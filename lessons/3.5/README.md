# Lesson 3.5: Realtime vs batch

The lesson's files, exactly as they were typed on camera (commit `Message Batches: submit, collect by custom_id, list a page at a time`, on top of Lesson 3.4).

| File | What it is |
|---|---|
| `quizforge/batch.py` | `custom_id()` (`L3.2` + `v4` → `L3_2-v4`), `requests()` (one request per lesson, the first request `generate_one` sends, no repair turns), `submit()` (creates the batch, logs it to `runs/batches.jsonl`), `collect()` (streams the results, matches each by `custom_id`, checks it with `validate.problems`) |
| `experiments/batch_submit.py` | Every library lesson as one batch, plus one request broken on purpose |
| `experiments/batch_collect.py` | The last batch: "Come back later" while it's processing; once it has ended, each result matched and checked, the order, and the price at the standard and batch rates |
| `experiments/listing.py` | The workspace's batches two at a time (`has_more`, then every page), and its files |

Run them from your `quiz-forge` project, with the environment active:

```bash
python -m experiments.batch_submit
python -m experiments.batch_collect   # "Come back later" until the batch has ended
python -m experiments.listing
```

One batch of 15 Claude Haiku 4.5 requests: about 5 cents. On camera: the batch ended in 285 s, 14 succeeded and the broken request errored (unbilled); the results came back in a different order from the requests; 8 of 14 passed the checks; $0.0521 at the batch price, $0.1042 at the standard price. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
