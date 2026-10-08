# Lesson 2.1: Production prompts

The lesson's files, exactly as they were typed on camera (commit `Prompts as files: v1, and v2 with tags, context, criteria and format`).

| File | What it is |
|---|---|
| `prompts/item_writer/v1.md` | The Lab 1 prompt from `quizforge/generate.py`, moved into a file as a template: the lesson id, the lesson, and one line asking for the questions |
| `prompts/item_writer/v2.md` | The production prompt: the lesson on top in `<lesson>` tags, then `<context>` (who reads the questions and why), `<instructions>` written as success criteria, and `<format>` (plain text, the exact layout) |
| `experiments/compare_prompts.py` | The checker: renders a prompt version for lesson L2.1, sends it N times to Claude Haiku 4.5, runs four checks in code on every reply (plain text, 8 options, no "all/none of the above", exact quotes), prints the totals and the answer letters, and saves the last reply to `runs/<version>-reply.md` |

Run it from your `quiz-forge` project (Lab 1), with the environment active:

```bash
python experiments/compare_prompts.py v1 5   # four checks in code on five replies; last reply in runs/v1-reply.md
python experiments/compare_prompts.py v2 5   # the same checks on version 2
```

Ten Claude Haiku 4.5 requests of about 3,900 input tokens each: about six cents in all. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
