# Lesson 1.4: Reading the response, and conversations

The lesson's files, exactly as they were typed on camera (commit `Read responses, handle stop reasons, keep a conversation`).

| File | What it is |
|---|---|
| `experiments/read_response.py` | One call; saves the whole message as JSON in `runs/message.json`, then prints each content block with its type, `stop_reason`, `usage` and the request id (`message._request_id`) |
| `experiments/stop_reasons.py` | `handle()`: a `match` on `stop_reason` with a case for every value. `ask()` sends one prompt four ways: plain, `max_tokens=15`, `stop_sequences=["B)"]`, and a forced tool call (`tool_choice`) |
| `experiments/chat.py` | A three-turn conversation: the `messages` list grows with every turn and is sent whole on every call. Then a fresh call without the history |

Run it from your `quiz-forge` project (Lessons 0.3 to 1.3), with the environment active and a `runs/` folder:

```bash
python experiments/read_response.py   # one text block, stop_reason: end_turn, usage, request id
python experiments/stop_reasons.py    # end_turn, max_tokens (15 out), stop_sequence, tool_use
python experiments/chat.py            # input tokens grow every turn; the fresh call can't quote turn one
```

The whole hands-on cost less than half a cent: nine small Claude Haiku 4.5 calls. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
