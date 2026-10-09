# Lesson 2.4: Structured output by design

The lesson's files, exactly as they were typed on camera (commit `A schema for questions, and as_data.py to compare ask, parse and a strict tool`).

| File | What it is |
|---|---|
| `quizforge/schema.py` | Two Pydantic designs for a question. `ListQuestion`: options in a list, each with a key, text, a `correct` flag and a `why`. `Question`: four required option fields `a` to `d`, an `answer` limited to A–D, a `why` and a `quote`. `ListQuestions` and `Questions` hold several |
| `experiments/as_data.py` | Writes two questions about lesson L2.1 four ways, N runs each, on Claude Haiku 4.5: `ask` (the schema pasted into the prompt, then `model_validate_json`), `list` and `fixed` (`messages.parse()` with each schema), and `tool` (a `strict: true` tool, forced with `tool_choice`). Per run it prints whether the reply was valid and whether every question had 4 options and 1 correct answer; per mode, the totals and the input tokens per call. Each reply is saved to `runs/as_data-<mode>-<run>.json` (`ask` replies also as `.txt`) |

The system prompt is `prompts/item_writer/system.md` from Lesson 2.2, with its `<format>` block removed in code: the layout now comes from the schema.

Run it from your `quiz-forge` project (Lesson 2.3), with the environment active:

```bash
python -m experiments.as_data 5 ask,list    # the schema pasted into the prompt vs parse() with a list of options
python -m experiments.as_data 5 fixed,tool  # four named option fields, then a strict tool
```

Twenty Claude Haiku 4.5 requests, each with a lesson of about 4,000 tokens: about twenty cents. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
