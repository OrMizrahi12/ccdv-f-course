# Lesson 1.2: Sampling and non-determinism

The two experiments from the lesson, exactly as they were typed on camera (commit `Add the sampling experiments`).

| File | What it is |
|---|---|
| `experiments/sampling.py` | One prompt, three ways: five calls to Claude Haiku 4.5 at the default temperature, five at temperature 0 (sent in `extra_body`), and one call to Claude Haiku 5.5 at temperature 0, which returns a 400 |
| `experiments/check_questions.py` | Asks Claude Haiku 5.5 five times for a multiple-choice question about the library's `L1.3.md`, as structured output (`messages.parse` with a Pydantic model), and checks properties: four options, an answer index in range, a question mark at the end |

`check_questions.py` reads `library/ccao-f/v1/L1.3.md`, the course library from Lesson 1.1 ([`library/ccao-f/v1/`](../../library/ccao-f/v1/)).

Run it from your `quiz-forge` project (Lesson 0.3), with the environment active:

```bash
python experiments/sampling.py          # 10 tiny Haiku 4.5 calls + 1 call that fails with a 400
python experiments/check_questions.py   # 5 Haiku 5.5 calls, about 6,100 input tokens each
```

On the recording machine: five different questions at the default temperature, the same question five times at temperature 0, `400: temperature is deprecated for this model.` on Haiku 5.5, then five `PASS` lines with five different questions. Your wording will differ; the 400 and the `PASS` lines are what should repeat. The whole hands-on cost about half a cent.
