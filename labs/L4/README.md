# Lab 4: a draft → check → revise agent, library search, item memory, human review

Claude Certified Developer – Foundations (CCDV-F) · Module 4: Context Engineering & Agents · the module checkpoint

In Module 4 you built the pieces one lesson at a time: a workflow and an agent measured on the same job (4.1); an agent loop with explicit stops and a person before the save (4.2); a fresh context per subtask (4.3); library sections scored with BM25 and sent as search results (4.4); subagents that start with a clean context (4.5); memory, Claude's own and yours (4.6); and the other ways to build an agent (4.7, 4.8). They still live in `experiments/` and in small modules. In this lab they become two Quiz Forge commands:

- `write`: an evaluator–optimizer job. A writer agent searches the library, drafts a lesson's questions and saves them to `bank/pending/`; a checker subagent, in a clean context, gives one verdict per question; the writer revises what isn't supported. The job stops for a reason your code chose.
- `review`: a person approves or declines the pending questions, one lesson at a time. Only a yes moves them to `bank/approved/`.

Underneath, item memory: every scenario already in the bank is remembered, and a new one that repeats it is refused by the save check, in code.

You pass the lab when the self-check tests pass and a real lesson goes through `write` and then `review`.

## What you start from

`starter/` is the `quiz-forge` project exactly as it stands after Lesson 4.8: Lab 3's commands, `quizforge/meter.py` and `docs/decisions.md` (4.1), `agent.py` and `writer_tools.py` (4.2), `retrieve.py` (4.4), `team.py` (4.5), `memory.py` (4.6), and the experiments of all four modules. If your own project is there too, keep working in it; if not, copy `starter/` to `~/projects/quiz-forge`, add your `.env` (see the course setup), and install the pinned `requirements.txt`.

`solution/` is a finished version to compare with when you're done; its `bank/` holds the reference run's two lessons. `tests/` holds the self-check.

## What you build

```
quiz-forge/
  config.toml      + [write] rounds, max_turns, max_usd and [bank] path, similar
  quizforge/
    search.py      search_library: lesson sections scored by retrieve.py's BM25, sent as search results with citations
    memory.py      + item memory: every stem in bank/pending/ and bank/approved/, overlap() and repeats()
    write.py       the job: the writer agent, the checker subagent, the save check, the stops
    review.py      the person's checkpoint: y to bank/approved/, n to bank/declined/
    sources.py     + shown(), verdicts(), supported(): review()'s pieces, now shared with the checker
    meter.py       + log=: every request of a job goes to runs/usage.jsonl
    cli.py         commands: write, review
  bank/            pending/ approved/ declined/ (committed to git; approved/ is written only by review)
  tests/           copied from labs/L4/tests
```

The rules that carry over from the module: start with the simplest thing that works, and add an agent where the path can't be fixed in advance; Claude asks and your code acts; every stop is a decision in code (done, a turn limit, a budget checked before each request, a person's no), never a request in a prompt; a person comes before the step that can't be undone; context is a budget, so a subtask gets a fresh context with only what it needs; retrieve the passages that matter, as search results Claude can cite; a subagent sees its brief and nothing else; a rule you know in advance is enforced in code, not left to Claude's memory.

## Steps

### 1. Library search (Lesson 4.4)
`quizforge/search.py`:
- `tool(library)`: one tool, `search_library`. Its description says what it does (the three best sections, as search results you can cite), how to use it (a few key words; search again with other words), and what it can't do (nothing outside the library). `query` is required; `lesson_id`, an enum of the library's ids, is optional.
- `search_library(library, query, lesson_id=None)`: reuse `retrieve.search` (BM25 over the `##` sections, Chapters and Sources skipped). Rank once across the whole library, then keep the given lesson's sections. Return the top three as `retrieve.as_results` blocks: `type: search_result`, a stable `source` such as `L1.3#the-three-tiers`, a title, the section's cleaned text, and `citations: {"enabled": true}`. An empty query or an unknown lesson id raises an instructive error.
- `section_of(library, lesson_id, quote)`: the section a quote comes from. A saved question keeps it, so its citation points at a passage, not just a lesson.

### 2. Item memory (Lesson 4.6)
`quizforge/memory.py` keeps the memory-tool handler and gains your own store. The rule is known in advance (no new scenario may repeat an old one), so code enforces it, as the own store did in Lesson 4.6:
- `known_stems(bank, skip=None)`: `(stem, where)` for every question in `bank/pending/` and `bank/approved/`. Declined questions aren't remembered. `skip` leaves out one file: the job's own draft, so a revision doesn't count as a repeat of itself.
- `overlap(a, b)`: shared words over all the words of both stems (Jaccard), with the search's own `words()`: common words dropped, plurals folded.
- `repeats(stems, known, limit)`: one problem line for each new stem whose overlap with a known stem, or with an earlier stem in the same save, is `limit` or more. The line says where the old stem is and what to do: `Write a different situation, not a reworded one`.
- The threshold is `similar = 0.4` in `config.toml`. We measured it on two real L1.3 stems, a few new situations and rewordings of them: different situations shared 3 to 26% of their words, close rewordings 51 to 76%. A scenario retold in mostly new words can still pass (31% in that test): word overlap is simple and explainable, and the person in `review` is the last check.

### 3. The job: draft, check, revise (Lessons 4.1, 4.2, 4.3, 4.5)
`quizforge/write.py`:
- The writer's tools: `search_library`, `check_quote` and `save_questions`. The writer gets no `read_lesson`: it loads the sections it needs, when it needs them.
- `save(cfg, job, lesson_id, questions)`: the draft's checkpoint, in code. The job's lesson only; then `validate.problems` (the count, four different options, a scenario that ends in a question, each quote word for word in the lesson); then `memory.repeats`. A failure raises `NotChecked` with every problem, and nothing is written. If the save passes, the questions, each with its `section`, go to `bank/pending/<lesson>.json`. Saving again replaces the job's draft.
- `brief(cfg, lesson_id)`: the first ask. The lesson's title and objectives from `MANIFEST.json`, how to use the tools, and the scenarios the bank already holds for this lesson. The save check still enforces it, whatever Claude remembers.
- `check(meter, cfg, job, limits)`: the checker subagent. It runs `agent.run` with `sources.REVIEW`, the `check_quote` tool only, and the questions as `sources.shown` formats them: a clean context, never the writer's conversation. One verdict per question; a missing verdict counts as not supported. It runs on `agent.run`, not on `sources.review`, so its requests get the same turn limit and budget check as the writer's.
- `revision(cfg, job)`: the next round's ask, in a fresh context. It holds the saved draft and the checker's verdicts: revise what isn't supported, keep the rest exactly as it is, save all of them again.
- `write(meter, cfg, lesson_id)`: up to `rounds` drafts. Writer and checker share one meter, and `agent.run` checks the meter's spend, so `max_usd` is the budget for the whole job. The job ends with `Result(stopped, who, rounds, job)`, and `stopped` is one of:
  - `done`: the checker supported every question;
  - `rounds limit`: still not supported after the last revision;
  - `turn limit`, `budget`, `cut off`, `refused`: from the writer's or the checker's run (`who` says which);
  - `not saved`: the writer ended its turn without a save that passed.
- A lesson that already has questions in `bank/pending/` isn't written again (`Waiting`: review them first). Nothing in `write.py` writes to `bank/approved/`.

### 4. The person's checkpoint (Lesson 4.2)
`quizforge/review.py`:
- `review(cfg, lessons=None, ask=input)`: one pending lesson at a time, in order. It shows each question, its options and key, its quote with the section, and the checker's verdict, with a warning if any question isn't supported. Then it asks.
- `y` (or `yes`) moves the lesson's questions to `bank/approved/<lesson>.json`, each with its verdict, model, prompt version and the date; `n` (or `no`) moves them to `bank/declined/`. Both add to what the file already holds. Any other answer leaves the lesson pending. With no one to answer (end of input), it stops, and nothing is decided.

### 5. The command line
`quizforge/cli.py`:
- `write --lesson L1.3`: checks the prompt lock before any request, then prints each round (the writer's turns, its searches and saves; the checker's turns and verdicts), the stop reason, where the questions wait, and the requests, input tokens and cost. Every request is logged to `runs/usage.jsonl` with the label `<lesson> write round <n> writer|checker`. The exit code is 0 only when the job stopped at `done`.
- `review [--lesson ...]`: every waiting lesson by default. When the answers come from a pipe, each one is echoed, so the transcript shows it.

### 6. Check your work
1. Copy `labs/L4/tests/` into your project as `tests/` and run:
   ```bash
   python -m pytest -q
   ```
   All 103 tests must pass. They need no API key: fake clients return the replies each test gives them. They check that the loop stops on each condition (done, turn limit, budget, a `Stop` from a tool, cut off, refused); that the job stops at the rounds limit and at a budget that counts writer and checker together; that a revision starts in a fresh context with the verdicts; that the checker's first request holds only the questions and `check_quote`; that a repeated stem is refused with an instructive message; that `write` never reaches `bank/approved/` and `review` moves nothing without a yes; and that a known query finds its section, as well-formed search result blocks.
2. Then the real thing, with the environment active:
   ```bash
   python -m quizforge write --lesson L1.3
   python -m quizforge write --lesson L2.2
   printf 'y\nn\n' | python -m quizforge review
   python -m quizforge cost
   ```
   The `printf` answers for you here: yes to the first lesson, no to the second. Day to day, run `python -m quizforge review` and answer yourself.
3. Commit: `git add .` and `git commit -m "Quiz Forge v0.4: a draft, check, revise agent with library search, item memory and human review"`. `bank/` goes into git; `runs/` stays out.

## What the reference solution printed (2026-10-09)

Claude Haiku 4.5. Your numbers, the questions and the path the writer takes will differ; the shape should not.

```
$ python -m quizforge write --lesson L1.3
L1.3 round 1: the writer drafts
  turn 1: tool_use, ['search_library', 'search_library', 'search_library'], $0.0031 so far
    search_library 'Haiku Sonnet Opus capabilities speed cost': L1.3#learning-outcomes, L1.3#key-terms, L1.3#the-rule-and-the-map
    search_library 'model tier tradeoff capability quality performance': L1.3#learning-outcomes, L1.3#the-rule-and-the-map, L1.3#key-terms
    search_library 'match job cost speed volume requirements': L1.3#learning-outcomes, L1.3#the-exam-link, L1.3#the-rule-and-the-map
  turn 2: tool_use, ['search_library', 'search_library'], $0.0093 so far
    search_library 'Haiku simple fast low cost high volume': L1.3#the-rule-and-the-map, L1.3#the-three-tiers, L1.3#the-exam-link
    search_library 'Opus complex reasoning capability accuracy': L1.3#the-rule-and-the-map, L1.3#the-exam-link, L1.3#learning-outcomes
  turn 3: tool_use, ['check_quote', 'check_quote', 'check_quote', 'check_quote'], $0.0191 so far
  turn 4: tool_use, ['save_questions'], $0.0301 so far
    save_questions: saved 2 questions to bank/pending/L1.3.json
  turn 5: end_turn, [], $0.0399 so far
L1.3 round 1: the checker, in a clean context
  turn 1: tool_use, ['check_quote', 'check_quote'], $0.0419 so far
  turn 2: end_turn, [], $0.0434 so far
  Question 1: supported
  Question 2: supported

stopped: done (checker, round 1 of 3)
2 question(s) in bank/pending/L1.3.json, 2 supported by the checker: waiting for review (python -m quizforge review --lesson L1.3)
7 request(s), 34,858 input tokens, $0.04335
$ python -m quizforge write --lesson L2.2
L2.2 round 1: the writer drafts
  turn 1: tool_use, ['search_library', 'search_library', 'search_library'], $0.0031 so far
    search_library 'task decomposition prompt chaining steps output': L2.2#learning-outcomes, L2.2#key-terms, L2.2#straight-from-the-source-(what-the-lesson-shows)
    search_library 'when to split one prompt checkpoint': L2.2#learning-outcomes, L2.2#the-idea, L2.2#the-exam-link
    search_library 'decision when to decompose': L2.2#description, L2.2#what-happened-in-the-hands-on, L2.2#straight-from-the-source-(what-the-lesson-shows)
  turn 2: tool_use, ['check_quote', 'check_quote', 'check_quote'], $0.0116 so far
  turn 3: tool_use, ['save_questions'], $0.0226 so far
    save_questions: ERROR not saved: question 2: scenario asks no question. Fix these and call save_questions again.
  turn 4: tool_use, ['save_questions'], $0.0341 so far
    save_questions: saved 2 questions to bank/pending/L2.2.json
  turn 5: end_turn, [], $0.0442 so far
L2.2 round 1: the checker, in a clean context
  turn 1: tool_use, ['check_quote', 'check_quote'], $0.0463 so far
  turn 2: end_turn, [], $0.0479 so far
  Question 1: supported
  Question 2: supported

stopped: done (checker, round 1 of 3)
2 question(s) in bank/pending/L2.2.json, 2 supported by the checker: waiting for review (python -m quizforge review --lesson L2.2)
7 request(s), 36,528 input tokens, $0.04791
$ printf 'y\nn\n' | python -m quizforge review
L1.3: 2 question(s), claude-haiku-4-5-20251001, prompt v4, job stopped: done (checker, round 1)

1. Your team processes 10,000 customer service emails per day, each requiring a brief, friendly acknowledgment. The emails are simple and straightforward. Which model should you start with?
   A) Opus, because it's the most capable and your customers deserve the best.
   B) Haiku, because these are short, simple jobs at high volume where speed and cost matter most.
   C) Sonnet, because it balances capability and cost evenly.
   D) Whichever is fastest on any given day, since volume is unpredictable.
   Answer: B. The job matches Haiku's profile: short, simple, high-volume work where speed and cost are the priorities. The lesson explicitly recommends Haiku for this type of job.
   Quote (L1.3#the-rule-and-the-map): "Short, simple jobs at volume, where speed and cost matter most"
   Checker: supported

2. A researcher needs to analyze one highly technical 100-page academic paper to identify potential flaws in the methodology. The accuracy of the analysis is critical, and time is not a constraint. Which model best fits this job?
   A) Haiku, to save tokens and complete the analysis quickly.
   B) Sonnet, as the everyday default that balances speed and capability.
   C) Opus, because complex reasoning is needed and accuracy outweighs the cost.
   D) Any model, since a single-use job doesn't multiply costs.
   Answer: C. This is a one-off job where quality and accuracy are everything. The lesson recommends Opus for scenarios where complex reasoning is required and being right outweighs cost considerations.
   Quote (L1.3#the-rule-and-the-map): "Complex reasoning, where being right outweighs the cost"
   Checker: supported

Approve these 2 question(s) for L1.3? [y/n] y
L1.3: approved, moved to bank/approved/L1.3.json
L2.2: 2 question(s), claude-haiku-4-5-20251001, prompt v4, job stopped: done (checker, round 1)

1. A marketing team asks you to analyze customer feedback from 200 emails, categorize the issues, identify the top complaints, and draft a response template—all in one prompt. You run it and get a template back. However, you're unsure whether the top complaints are actually the most common or if there's an error in the categorization that went undetected. What should you do?
   A) Run the same prompt again on a more capable model to double-check the accuracy.
   B) Ask Claude to add 'think step by step' before generating the template.
   C) Decompose the task: ask Claude to categorize and count the issues first, review those results, then ask for the response template based on the verified counts.
   D) Send each email individually to Claude for categorization, then combine the results manually.
   Answer: C. A chain with a checkpoint after counting lets you verify the intermediate output before the final step, catching categorization errors before they affect the template. Option A wastes a bigger model on the same blind spot. Option B relies on thinking, which happens invisibly inside one reply. Option D splits too far—categorization depends on seeing all emails together.
   Quote (L2.2#the-idea): "Between two steps you can read one, fix it, and only then continue."
   Checker: supported

2. A project manager needs to extract action items from meeting notes, summarize each action's dependencies, order them by priority, and create a timeline. She's uncertain whether to send this all at once or break it into steps. She knows the task is complex and multistage. Which approach should she choose?
   A) Send it all in one prompt; Claude's thinking will handle the reasoning internally.
   B) Break it into separate prompts where she can inspect the extracted actions and dependencies before Claude creates the timeline.
   C) Use numbered steps inside one prompt to keep everything in order without extra waiting.
   D) Ask Claude to think harder by increasing effort settings before sending the single prompt.
   Answer: B. Explicit chaining with a checkpoint is useful when you need to inspect intermediate outputs and verify accuracy between steps. Thinking happens invisibly within one reply (A). Numbered steps in one prompt don't let you steer midway (C). Effort settings don't replace the value of seeing and validating in-between results (D).
   Quote (L2.2#straight-from-the-source-(what-the-lesson-shows)): "Claude handles most multistep reasoning internally; explicit chaining is still useful when you need to inspect intermediate outputs."
   Checker: supported

Approve these 2 question(s) for L2.2? [y/n] n
L2.2: declined, moved to bank/declined/L2.2.json

L1.3 approved, L2.2 declined
$ python -m quizforge cost
14 requests, $0.09126
  L1.3   $0.04335
  L2.2   $0.04791
```

Both lessons passed the checker in the first round, so neither needed a revision. The tests cover that path, the rounds limit and the budget. In L2.2 the save check refused the first save (a scenario that asked no question); the writer fixed it and saved again on its next turn. The checker's requests carried 1,140 to 1,493 input tokens, while the writer's grew from about 1,900 to 8,900 over its five turns: the checker saw the questions and its one tool, not the writer's searches. After `review`, `bank/approved/L1.3.json` holds L1.3's two questions, `bank/declined/L2.2.json` holds L2.2's, and `bank/pending/` is empty. Together the two jobs cost 9 cents.

## Going further (optional)
- Run `write --lesson L1.3` again. The brief now lists the approved scenarios: do the new ones differ? Lower `similar` to 0.25 and run it once more: what does the save check refuse, and is it right?
- Give the checker `search_library` too, so it can read the passage around a quote. Does it change a verdict? What does it cost per job?

## Sources (as checked for Lessons 4.1 to 4.6)
- Building effective agents: https://www.anthropic.com/engineering/building-effective-agents
- Build a tool-using agent: https://platform.claude.com/docs/en/agents-and-tools/tool-use/build-a-tool-using-agent
- Tool runner: https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-runner
- Effective context engineering for AI agents: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Introducing Contextual Retrieval: https://www.anthropic.com/engineering/contextual-retrieval
- Search results: https://platform.claude.com/docs/en/build-with-claude/search-results
- How we built our multi-agent research system: https://www.anthropic.com/engineering/multi-agent-research-system
- Memory tool: https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool
