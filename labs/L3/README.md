# Lab 3: a source-checking tool, PDF input with citations, overnight batch generation

Claude Certified Developer – Foundations (CCDV-F) · Module 3: Tools & the Claude API in Depth · the module checkpoint

In Module 3 you built the pieces one lesson at a time: a tool Claude selects from its description, and the loop that runs it (3.1); every call of a reply run together, failures sent back as `is_error`, thinking blocks kept (3.2); an approval step and the difference between your tools and Anthropic's (3.3); PDFs with citations and the Files API (3.4); and Message Batches matched by `custom_id` (3.5). They still live in `experiments/`. In this lab they become three Quiz Forge commands:

- `check`: Claude reviews a lesson's saved questions with **your** source-checking tool, `check_quote`, which says whether each quote really is in the lesson.
- `evidence`: the lesson's PDF, uploaded once with the Files API, and the passages, with their pages, that support each answer (citations).
- `batch submit` and `batch collect`: every lesson as one Message Batch at half the price, results matched by `custom_id`, and a second pass through the realtime repair loop for the replies that fail the checks.

You pass the lab when the self-check tests pass and a real batch is collected, checked and saved.

## What you start from

`starter/` is the `quiz-forge` project exactly as it stands after Lesson 3.5: Lab 2's `generate` command, `quizforge/tools.py`, `files.py` and `batch.py`, and the experiments of all three modules. If your own project is there too, keep working in it; if not, copy `starter/` to `~/projects/quiz-forge`, add your `.env` (see the course setup), and install the pinned `requirements.txt`.

`solution/` is a finished version to compare with when you're done. `tests/` holds the self-check.

## What you build

```
quiz-forge/
  quizforge/
    sources.py     check_quote, its tool definition, harness dispatch, run_all, and review(): a manual tool loop
    evidence.py    the lesson's PDF by file_id, with citations: each supporting passage and its page
    repair.py      write_checked(..., history=...): the loop can start from a reply it didn't request
    batch.py       + version_of, last_sent, collect() that yields the reply text, finish() and the realtime second pass
    cli.py         commands: check, evidence, batch submit, batch collect
  tests/           copied from labs/L3/tests
```

The rules that carry over from the module: Claude asks and your code acts; a tool's description says what it does, when to use it, and what it can't do; every `tool_use` gets its `tool_result`, all of one reply's results in one user message; a failure goes back as `is_error` with what to do next; citations come from the document, not from a request in the prompt; a file goes up once; batch results are matched by `custom_id`, never by position.

## Steps

### 1. The source-checking tool (Lessons 3.1, 3.2)
`quizforge/sources.py`:
- `check_quote(library, lesson_id, quote)`: `found in L1.3` when the quote's words appear in the cleaned lesson (case and punctuation ignored, like `validate.problems`); otherwise `not found in L1.3. Closest sentence: '...'`, the lesson's sentence that shares the most words with the quote, so Claude can correct it.
- An unknown lesson id or a quote under four words raises `NotChecked` with an instructive message: what was wrong and what to send instead.
- `tools(library)`: one tool, `check_quote`. Its description says what it does, when to use it (once per question, several at once), and what it can't do (nothing outside the library). `lesson_id` is an enum of the library's ids; both inputs are required.
- `run(library, call)`: harness dispatch. A failure, including a missing input, becomes `{"type": "tool_result", "tool_use_id": ..., "content": <the message>, "is_error": true}`, never an empty string.
- `run_all(library, calls)`: every call of one reply at once, the results in the order of the calls, for one user message.
- `review(client, cfg, lesson_id, questions)`: a manual loop. Claude gets the questions (the quote, the key) and a short system prompt; it calls `check_quote`, your code runs the calls and sends all the results back in one user message, until Claude replies with one line per question: `Question <n>: supported` or `Question <n>: not supported: <reason>`. Stop after a few turns.

### 2. Evidence from the PDF, with citations (Lesson 3.4)
`quizforge/evidence.py`:
- `pdf_of(cfg, lesson_id)`: `library/ccao-f/v1-docs/<lesson>.pdf`; a clear `FileNotFoundError` if there's none.
- `evidence(client, cfg, lesson_id, questions)`: `files.file_id` uploads the PDF once and reuses its `file_id` afterwards. The request has one `document` block (`source: {"type": "file", "file_id": ...}`, a title, `citations: {"enabled": true}`) and the questions with their keys. Return each text block with its citations as `(cited_text, start_page_number)`. No `output_config`: citations and structured outputs can't be combined.

### 3. Batch, then a realtime second pass (Lessons 3.5, 2.5)
- `repair.write_checked` gets `history=None`. Given a conversation that already holds the first reply and its problems (`[ask, reply, fix]`), the loop goes on from attempt 2.
- `batch.collect` yields `(custom_id, outcome, problems, text, usage)` for each result, matched to its lesson by `custom_id`.
- `batch.finish(cfg, prices, client, ...)` turns one result into a saved result, with `via`:
  - passed: saved at the batch price (half of `usage.cost`), logged with the label `<lesson> <version> batch`;
  - failed the checks: the batch reply and its problems go through the repair loop in realtime, counted and logged like any repair;
  - errored: saved to `runs/rejected/` with the error type, not billed;
  - expired or canceled: sent again in realtime with `generate_one`.

### 4. The command line
`quizforge/cli.py`:
- `check --lesson L1.3`: runs `review` on the saved questions, prints the verdicts, the number of `check_quote` calls and the cost, and stores the verdicts in the item file. The exit code is 0 only if every question is supported.
- `evidence --lesson L1.3`: whether the PDF was uploaded or reused, the input tokens and cost, and each supporting passage with its page.
- `batch submit [--lesson ...]`: every lesson in the library by default. Prints the batch id, the number of requests, and the first and last `custom_id`.
- `batch collect [--batch ID]`: the last batch by default. If it hasn't ended: `Come back later.` (exit code 2). Once it has: how long it took, the counts, then one line per lesson and a total.
- Like `generate`, `batch` checks the prompt lock before any request.

### 5. Check your work
1. Copy `labs/L3/tests/` into your project as `tests/` and run:
   ```bash
   python -m pytest -q
   ```
   All 61 tests must pass. They need no API key: fake clients return the replies each test gives them, including batch results in a shuffled order.
2. Then the real thing, with the environment active:
   ```bash
   python -m quizforge batch submit --lesson L1.3 --lesson L2.1 --lesson L2.2 --lesson L3.1 --lesson L3.2 --lesson L0.3
   python -m quizforge batch collect
   ```
   Run `batch collect` again until the batch has ended (most end within an hour, ours in a few minutes). Then:
   ```bash
   python -m quizforge check --lesson L1.3
   python -m quizforge evidence --lesson L1.3
   python -m quizforge evidence --lesson L1.3
   python -m quizforge cost
   ```
3. Commit: `git add .` and `git commit -m "Quiz Forge v0.3: a source-checking tool, PDF evidence with citations, batch generation"`. `runs/` stays out of git.

## What the reference solution printed (2026-10-09)

Claude Haiku 4.5. Your numbers, the questions and the order of the results will differ; the shape should not.

```
$ python -m quizforge batch submit --lesson L1.3 --lesson L2.1 --lesson L2.2 --lesson L3.1 --lesson L3.2 --lesson L0.3
msgbatch_01XBnLz5bxzeLLmFgSHHKu7R: 6 requests (L1_3-v4 ... L0_3-v4), in_progress
$ python -m quizforge batch collect
msgbatch_01XBnLz5bxzeLLmFgSHHKu7R: in_progress, 6 processing. Come back later.
$ python -m quizforge batch collect
msgbatch_01XBnLz5bxzeLLmFgSHHKu7R ended in 377 s: 6 succeeded, 0 errored, 0 expired, 0 canceled
L3.2  batch reply: question 1: quote not in the lesson: repairing in realtime
L3.2  attempt 2: 6092 input tokens, pass
L2.1  batch reply: expected 2 questions, got 4: repairing in realtime
L2.1  attempt 2: 5741 input tokens, pass
L0.3  batch reply: question 2: quote not in the lesson: repairing in realtime
L0.3  attempt 2: 4064 input tokens, question 2: quote not in the lesson
L0.3  attempt 3: 4434 input tokens, pass

L0.3  passed   batch, 3 attempt(s), $0.01472
L1.3  passed   batch, 1 attempt(s), $0.00378
L2.1  passed   batch, 2 attempt(s), $0.01340
L2.2  passed   batch, 1 attempt(s), $0.00387
L3.1  passed   batch, 1 attempt(s), $0.00386
L3.2  passed   batch, 2 attempt(s), $0.01239
6 of 6 passed, $0.05202
$ python -m quizforge check --lesson L1.3
  turn 1: tool_use, 2 check_quote call(s)
    check_quote: 'found in L1.3'
    check_quote: 'found in L1.3'
  turn 2: end_turn, 0 check_quote call(s)
Question 1: supported
Question 2: supported
2 check_quote call(s), 2 request(s), $0.00349
$ python -m quizforge evidence --lesson L1.3
L1.3.pdf: uploaded, 13328 input tokens, $0.01399
  The lesson's selection rule starts with the smallest tier that can clearly do the job, and
    [page 2] 'Start with the smallest tier that can clearly do the job.'
    [page 2] 'The job Start with\r\nShort, simple jobs at volume, where speed and cost matter mo'
  A one-off job where accuracy is everything pulls toward the most capable model, which is A
    [page 2] 'A one-off job where accuracy is everything pulls the other way.'
    [page 3] 'It\'s Anthropic\'s\r\ncapability-first case ("Applications where accuracy outweighs '
$ python -m quizforge evidence --lesson L1.3
L1.3.pdf: reused by file_id, 13328 input tokens, $0.01401
  Start with the smallest tier that can clearly do the job, and volume multiplies every gap,
    [page 2] 'Start with the smallest tier that can clearly do the job.'
    [page 2] 'The job Start with\r\nShort, simple jobs at volume, where speed and cost matter mo'
  Complex reasoning where being right outweighs the cost, and a one-off job where accuracy i
    [page 2] 'The job Start with\r\nShort, simple jobs at volume, where speed and cost matter mo'
    [page 2] 'A one-off job where accuracy is everything pulls the other way.'
    [page 3] 'It\'s Anthropic\'s\r\ncapability-first case ("Applications where accuracy outweighs '
$ python -m quizforge cost
10 requests, $0.05202
  L0.3   $0.01472
  L1.3   $0.00378
  L2.1   $0.01340
  L2.2   $0.00387
  L3.1   $0.00386
  L3.2   $0.01239
```

Three of the six batch replies failed the checks (a quote not in the lesson, four questions instead of two) and were repaired in realtime, starting from the batch reply. `check` made one `check_quote` call per question, both in the same reply. The second `evidence` reused the `file_id`: nothing was uploaded, and the input tokens were the same.

## Going further (optional)
- Give `review` a second tool, `read_lesson`, so Claude can read the passage around a quote that was found. Does it change any verdict? What does it cost?
- Run `batch submit` with no `--lesson` (the whole library) and compare the batch total with what `generate` would have cost for the same lessons.

## Sources (as checked for Lessons 3.1 to 3.5)
- How tool use works: https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works
- Define tools: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
- Handle tool calls: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls
- Parallel tool use: https://platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use
- Citations: https://platform.claude.com/docs/en/build-with-claude/citations
- PDF support: https://platform.claude.com/docs/en/build-with-claude/pdf-support
- Files API: https://platform.claude.com/docs/en/build-with-claude/files
- Batch processing: https://platform.claude.com/docs/en/build-with-claude/batch-processing
