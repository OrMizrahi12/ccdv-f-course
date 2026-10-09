# Lesson 2.3: Zero-, one- and multi-shot

The lesson's files, exactly as they were typed on camera (commit `Three examples in examples.md, and shots.py to measure them`).

| File | What it is |
|---|---|
| `prompts/item_writer/examples.md` | Three examples of the exact question layout from `system.md`, each in `<example>` tags. They are about a **different** lesson (choosing a model tier and effort), with answers C, A and D, so they show the shape of a question without lending this lesson's content |
| `experiments/shots.py` | Writes two questions about lesson L2.1 with 0, 1 or 3 examples from a file, N runs each, on Claude Haiku 4.5. Per run it checks, in code: **exact quotes** (every `Quote:` line appears word for word in the lesson) and **copied** (any six words in a row, on one line, shared with an example). Per setting it prints the totals, the input tokens per call and the answer letters. Each reply is saved to `runs/shots-<k>-<run>.md` |
| `experiments/same_lesson.md` | One example about lesson L2.1 itself: the copy trap |

Run it from your `quiz-forge` project (Lesson 2.2), with the environment active:

```bash
python experiments/shots.py 5 0,1,3 prompts/item_writer/examples.md   # 0, 1 and 3 examples, 5 runs each
python experiments/shots.py 5 1 experiments/same_lesson.md             # the copy trap: an example about the same lesson
```

Twenty Claude Haiku 4.5 requests, each with a lesson of about 4,000 tokens: about fifteen cents. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
