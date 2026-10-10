# Lesson 5.6: Claude Code in the pipeline

The reviewer files from the lesson, as they were committed on camera (merged to `main` as `b06cf19`).

| File | What it is |
|---|---|
| `reviewer/verdict.schema.json` | the JSON schema each review must fill: `item_id`, `verdict` (`pass` or `fail`), `reasons`, `rule` |
| `scripts/ci_review.sh` | reviews every pending item a branch adds or changes, with `claude -p`, the schema and the plugin's checklist skill; one table row per file; exits 1 if any review fails |
| `.github/workflows/review.yml` | runs the tests, then the review, on pull requests that touch the bank, the code, the plugin, the scripts or the reviewer |

Start from your `quiz-forge` project at the Lesson 5.5 state. Try the structured review locally first:

```bash
claude -p "Review bank/pending/L2.1.json with the quiz-forge-tools write-exam-item checklist." \
  --model claude-haiku-4-5 --permission-mode dontAsk --allowedTools "Read,Glob,Grep,Skill" --disallowedTools Agent \
  --max-turns 12 --output-format json --json-schema "$(cat reviewer/verdict.schema.json)" | jq .structured_output
```

For CI, push the repository to GitHub, add your API key as a repository secret named `ANTHROPIC_API_KEY`, and open a pull request that changes a pending item.

Cost: locally, `claude -p` runs on your Claude subscription. In CI it uses API credit: on camera the reviews cost about $0.50 in total across three runs. The script caps each review with `--max-budget-usd 0.40`. The student kit has the full steps for Windows, macOS and Linux.
