# Lesson 7.5: Prompt caching in depth

The code from the lesson, as it was committed on camera (`cd0d4af` and `8f28704`).

| File | What it is |
|---|---|
| `experiments/prefix_size.py` | Each lesson's cacheable prefix, counted with `count_tokens` (free), against Claude Haiku 4.5's 4,096-token minimum. |
| `experiments/cache_lab.py` | Six scenes on real requests: no cache, the breakpoint on the changing block, a stable prefix, a lesson under the minimum, a timestamp in the system prompt, another model. Each row shows input, cache write, cache read, output, cost and the miss reason. |
| `experiments/ttl_math.py` | Arithmetic only: the same prefix used 4 times at gaps of 2, 10, 30 and 90 minutes, with no cache, the 5-minute cache and the 1-hour cache. |
| `tests/test_caching.py` | Offline: where the breakpoint goes, and what writes and reads cost. A live self-check is marked `integration`. |
| `lesson.diff` | The changes to files you already have: the 1-hour write price in `pricing.toml`; 5-minute writes, 1-hour writes and reads priced separately in `usage.py`; every input token counted in `meter.py`; the lesson as a cached block in `generate.py`; cache reads printed per attempt in `repair.py`. Apply with `git apply lesson.diff`. |

Start from your `quiz-forge` project at the end of Lesson 7.4. Copy the files in, then:

```bash
git apply lesson.diff
.venv/bin/python -m experiments.prefix_size
.venv/bin/python -m experiments.cache_lab 3
.venv/bin/python -m experiments.ttl_math
.venv/bin/python -m pytest -q
.venv/bin/python -m pytest -q -m integration -k cache
```

Run `cache_lab` with each scene number, `1` to `6`, to see the lesson's table. Start cold: the cache is shared by everything that uses your key, so a run in the last 15 minutes can leave an entry that scene 2 reads. Cost: the six scenes come to about $0.11 (Claude Haiku 4.5; scene 6 adds one Claude Haiku 5.5 request), and part 2 about $0.02. The student kit has the full steps in the lesson's order, and what the lesson printed.
