# Lesson 1.1: How an LLM produces text

The two experiments from the lesson, exactly as they were typed on camera (commit `Add the course library and token experiments`).

| File | What it is |
|---|---|
| `experiments/count_tokens.py` | Counts a file's tokens on Claude Haiku 4.5 and Claude Haiku 5.5 with the free token counting API; prints characters, tokens, characters per token and the share of each context window |
| `experiments/next_token.py` | Streams a reply from Claude Haiku 4.5 with `max_tokens=20`, a bar (`\|`) after each piece, then the stop reason and the usage |

The library the lesson counts is in [`library/ccao-f/v1/`](../../library/ccao-f/v1/). Copy that folder into your `quiz-forge` project as `library/ccao-f/v1/`.

Run it from your `quiz-forge` project (Lesson 0.3), with the environment active:

```bash
python experiments/count_tokens.py library/ccao-f/v1/L1.3.md   # free: token counting
python experiments/next_token.py                               # one Haiku 4.5 call, at most 20 output tokens
```

On the recording machine: 15,287 characters; 4,351 tokens on Haiku 4.5, 5,772 on Haiku 5.5. The streamed text differs each run but always stops with `stop_reason: max_tokens`.
