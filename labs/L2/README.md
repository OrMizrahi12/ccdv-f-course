# Lab 2: versioned prompts, schema-validated items, a repair loop

Claude Certified Developer – Foundations (CCDV-F) · Module 2: Prompt Engineering & Output Handling · the module checkpoint

In Module 2 you built the pieces one lesson at a time: prompts as numbered files (2.1), the lesson kept apart from the rules (2.2), a schema for questions (2.4), checks in code and a repair loop with a limit (2.5), a sanitized lesson and a measured v4 (2.6). They still live in `experiments/`. In this lab they become Quiz Forge's `generate` command. It loads the prompt version named in `config.toml`, and refuses one that changed after it was measured. It cleans the lesson and puts it inside its tags. It gets the questions back as schema-valid JSON, checks them in code and sends the problems back, with a retry limit. Each attempt is cost-checked and logged. Questions that pass are saved; a lesson that never passes is set aside with its problems, never saved as if it were fine.

You pass the lab when the self-check tests pass and a real run saves checked questions.

## What you start from

`starter/` is the `quiz-forge` project exactly as it stands after Lesson 2.6: Lab 1's CLI, `quizforge/schema.py`, `validate.py`, `repair.py` and `sanitize.py`, the prompt versions `v1` to `v4` with their `CHANGELOG.md`, and the experiments of both modules. If your own project is there too, keep working in it; if not, copy `starter/` to `~/projects/quiz-forge`, add your `.env` (see the course setup), and install the pinned `requirements.txt`.

`solution/` is a finished version to compare with when you're done. `tests/` holds the self-check.

## What you build

```
quiz-forge/
  config.toml              + [prompt] folder and item_writer; retries; rejected_dir
  prompts/item_writer/
    LOCK                   one line per measured version: the version and the SHA-256 of its text
  quizforge/
    config.py              reads the new settings
    prompts.py             lists, locks and loads prompt versions
    repair.py              the Lesson 2.5 loop, with a hook before and after each request
    generate.py            one lesson, or several at once: locked prompt, clean lesson, checked JSON, repairs
    cli.py                 commands: generate (--prompt), show, prompts (--lock), model, cost
  tests/                   copied from labs/L2/tests
```

The rules that carry over from the module: a measured prompt is never edited, only followed by the next version; the lesson is data, cleaned and kept inside its tags; the format comes from the schema; code decides whether the questions pass; a repair has a limit; every request is counted first and logged after.

## Steps

### 1. Settings (Lessons 2.1, 2.5)
Add to `config.toml`:
- a `[prompt]` table: `folder = "prompts/item_writer"` and `item_writer = "v4"`;
- in `[generate]`: `retries = 2` and `rejected_dir = "runs/rejected"`.

Give `Config` in `quizforge/config.py` the fields `prompts` (a path), `prompt`, `retries` and `rejected_dir` (a path).

### 2. Lock the prompt versions (Lesson 2.6)
`quizforge/prompts.py`:
- `versions(folder)`: the `v<number>.md` files, sorted by number, so `v10` comes after `v9`. `system.md` isn't a version.
- `digest(text)`: the SHA-256 of the text.
- `locked(folder)`: the `LOCK` file as a dict, version to hash; empty if there's no file.
- `lock(folder, version)`: appends `<version> <hash>` to `LOCK`. A version that's already locked raises `PromptChanged`.
- `load_prompt(folder, version)`: the text. An unknown version raises `ValueError`, naming the ones that exist. A version that isn't locked, or no longer matches its hash, raises `PromptChanged`.
- `for_schema(text)`: the text without its `<format>` section, cut the way `experiments/measure.py` cuts it, so the prompt you send is the prompt you measured.

Then lock the four versions you have: `python -m quizforge prompts --lock v1`, and the same for `v2`, `v3`, `v4` (after step 5). Commit `LOCK`.

### 3. Hooks in the repair loop (Lesson 2.5)
`write_checked` in `quizforge/repair.py` gets two keyword arguments, so a repair is treated like the first attempt:
- `before(messages)`: called before every request (the cost check goes here).
- `after(attempt, message, seconds)`: called after every reply, before any check (the usage log goes here).

`RepairFailed` keeps the last problems and the last reply text, as `found` and `text`, so a rejected lesson can be saved with them.

### 4. Generate (Lessons 2.2, 2.4, 2.5, 2.6, 1.8)
`quizforge/generate.py`:
- `user_turn(lesson_id, text, questions)`: `<lesson id="L1.3">`, the cleaned lesson, `</lesson>`, then `Write 2 questions.`
- `OverLimit`: raised by the `before` hook when `count_tokens` plus `max_tokens` could cost more than `max_request_usd`. Nothing is sent.
- `generate_one(cfg, prices, lesson_id, version=None, client=None, out=print)`: loads the locked version (`for_schema`), cleans the lesson, runs `write_checked` with `retries` from the config, and logs each attempt as `<lesson> <version> attempt <n>`. It returns a dict with `lesson`, `prompt`, `model`, `status` (`passed`, `rejected`, `refused` or `over limit`), `attempts`, `cost` and `path`. Questions that pass go to `runs/items/<lesson>.json` with their questions; anything else goes to `runs/rejected/<lesson>.json` with its `problems` (and the last reply, for a rejection). A new result replaces the lesson's old one in the other folder. The quote check runs against the cleaned lesson, the text Claude actually saw.
- `generate_many(...)`: several lessons, at most `concurrency` at once. The repair loop is a plain loop, so each lesson runs with `asyncio.to_thread` under an `asyncio.Semaphore`, and the client is shared.

### 5. The command line
`quizforge/cli.py`:
- `generate --lesson L1.3 [--lesson ...] [--prompt v3]`: checks the prompt first (a changed, unlocked or unknown version stops here, before any request), then the model, then generates. It prints one line per lesson and a total. The exit code is 0 only if every lesson passed.
- `show --lesson L1.3`: the saved questions, readable, or the problems of a rejected lesson.
- `prompts`: each version, `locked`, `unlocked` or `CHANGED`, with its `Measured:` line from `CHANGELOG.md`, and a star on the one in `config.toml`. `prompts --lock v5` locks a version.
- `model` and `cost` as in Lab 1.

### 6. Check your work
1. Copy `labs/L2/tests/` into your project as `tests/` and run:
   ```bash
   python -m pytest -q
   ```
   All 42 tests must pass. They need no API key: a fake client returns the replies each test gives it.
2. Then the real thing, with the environment active:
   ```bash
   python -m quizforge prompts
   python -m quizforge generate --lesson L1.3
   python -m quizforge generate --lesson L2.1 --lesson L2.2 --lesson L3.1 --lesson L3.2
   python -m quizforge show --lesson L1.3
   python -m quizforge cost
   ```
3. Prove the lock works: add a line to `prompts/item_writer/v4.md`, run `python -m quizforge prompts` and `generate` again, then restore the file with `git checkout prompts/item_writer/v4.md`.
4. Commit: `git add .` and `git commit -m "Quiz Forge v0.2: locked prompt versions, schema-checked items, a repair loop"`. `runs/` stays out of git.

## What the reference solution printed (2026-10-09)

```
$ python -m quizforge prompts
  v1   locked
  v2   locked
  v3   locked   pass 11/18; quote not in the lesson 6; expected 2 questions, got 3 2
* v4   locked   pass 13/18; quote not in the lesson 4; scenario asks no question 3
(* in config.toml)

$ python -m quizforge generate --lesson L1.3
claude-haiku-4-5-20251001 | active | retires not scheduled
L1.3  attempt 1: 5243 input tokens, expected 2 questions, got 3 (+1 more)
L1.3  attempt 2: 5941 input tokens, pass

L1.3: passed after 2 attempt(s), $0.01664 -> runs/items/L1.3.json
1 of 1 passed, $0.01664

$ python -m quizforge generate --lesson L2.1 --lesson L2.2 --lesson L3.1 --lesson L3.2
claude-haiku-4-5-20251001 | active | retires not scheduled
  0.00 s  start  L2.1
  0.02 s  start  L2.2
  0.02 s  start  L3.1
L3.1  attempt 1: 5093 input tokens, pass
  8.05 s  passed L3.1  1 attempt(s)  $0.00717
  8.05 s  start  L3.2
L2.2  attempt 1: 4898 input tokens, pass
  8.56 s  passed L2.2  1 attempt(s)  $0.00742
L2.1  attempt 1: 4655 input tokens, pass
  9.42 s  passed L2.1  1 attempt(s)  $0.00741
L3.2  attempt 1: 5618 input tokens, pass
 15.22 s  passed L3.2  1 attempt(s)  $0.00769
...
4 of 4 passed, $0.02969, 15.22 s on the clock

$ python -m quizforge cost
6 requests, $0.04633
```

With a line added to `v4.md`, `prompts` showed `* v4   CHANGED` and `generate` stopped before any request: `v4.md changed after it was locked: restore it, and put the change in the next version`. `--prompt v9` stopped with `unknown prompt version 'v9'; item_writer has v1, v2, v3, v4`.

Your questions, token counts and timings will differ, and so will which lessons need a repair: v4 passed 13 of 18 runs when it was measured, so expect a repair now and then, and an occasional rejection. What should hold: no lesson is saved to `runs/items/` unless it passed the checks; a repair's request carries the problems and is logged like the first attempt; three lessons start at once and the fourth starts when one finishes; a changed prompt never reaches the API. The whole lab costs about five cents on Claude Haiku 4.5.

## Going further (optional)
- Write `v5`: one change aimed at v4's biggest failure kind. Measure it with `python -m experiments.measure v5 6`, record it in `CHANGELOG.md`, lock it, and point `config.toml` at it only if it measured better.
- Set `retries = 0` and run the five lessons again: how many would you have shipped without the repair loop, and what did the repairs cost?

## Sources (as checked for Lessons 2.4 to 2.6)
- Structured outputs (JSON outputs, SDK schema transformation, invalid outputs: refusals and `max_tokens`): https://platform.claude.com/docs/en/build-with-claude/structured-outputs
- Handling stop reasons: https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons
- Define success criteria and build evaluations: https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
- Mitigate jailbreaks and prompt injections: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks
- Token counting: https://platform.claude.com/docs/en/build-with-claude/token-counting
- Pricing: https://platform.claude.com/docs/en/about-claude/pricing
