# Lesson 1.5: Streaming, async and websockets

The lesson's files, exactly as they were typed on camera (commit `Stream replies, resume a broken stream, run calls at once`).

| File | What it is |
|---|---|
| `experiments/stream_request.json` | Lesson 1.3's `request.json` with one more field: `"stream": true` |
| `experiments/stream.py` | `client.messages.stream(...)`: prints `text_stream` as it arrives, times the first text, then `get_final_message()` for the assembled message (`stop_reason`, blocks, output tokens) |
| `experiments/resume.py` | A stream that breaks on purpose after 13 chunks (a stand-in `ConnectionError`), keeps the partial text, and resumes by sending it back as the start of the assistant turn (the path for Claude 4.5 and earlier) |
| `experiments/many.py` | `AsyncAnthropic`, `asyncio.Semaphore(3)` and `asyncio.gather` over six topics: at most three calls in flight, each one done lets the next start |

Run it from your `quiz-forge` project (Lessons 0.3 to 1.4), with the environment active:

```bash
python experiments/stream.py   # the text arrives as it's written; time to first text, then the final message
python experiments/resume.py   # breaks after 13 chunks on purpose, then resumes from the partial text
python experiments/many.py     # six topics, at most three calls in flight
```

The whole hands-on cost less than a cent: ten small Claude Haiku 4.5 calls. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
