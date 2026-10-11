# Lesson 6.4: Built-in tool, custom tool, Skill or MCP

The code from the lesson, as it was committed on camera (`4bb8ef1`).

| File | What it is |
|---|---|
| `experiments/tool_choice.py` | Measures what each option adds to one request on your writer model: the custom `check_quote` tool, the library server's three MCP tools, the built-in web fetch, a Skill's description and its whole `SKILL.md`. `count_tokens` is free; the web-fetch row needs two real 16-token requests, because `count_tokens` doesn't count server tools. |
| `docs/decisions.md` | The project's decision log. Section 2 is this lesson's: the four rules, the measured line, and the Need / Choice / Deciding constraint / What we give up table. |

Start from your `quiz-forge` project at the Lesson 6.3 state, with the Lesson 5.5 plugin in `plugin/quiz-forge-tools/`. Copy the files in, then:

```bash
.venv/bin/python -m experiments.tool_choice | tee runs/tool_choice.txt
claude -p "/context" | grep -A 15 "by category"
```

Cost: a fraction of a cent per run on Haiku 4.5 (only the two web-fetch requests are billed). On the recording machine: custom tool +767 tokens, the three MCP tools +878, built-in web fetch +1021, a Skill's description +44, all of `SKILL.md` +440. Your numbers depend on your model, your tools and your Claude Code version. The student kit has the full steps for Windows, macOS and Linux.
