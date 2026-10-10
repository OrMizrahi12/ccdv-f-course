# Lesson 4.6: Agent memory

The lesson's files, exactly as they were typed on camera (commit `Agent memory: the memory tool with a safe handler vs our own store, three sessions, measured`, on top of Lesson 4.5).

| File | What it is |
|---|---|
| `quizforge/memory.py` | The client-side handler for Claude's memory tool (`memory_20250818`): one folder per scope, any path that resolves outside it refused, the six commands, and every file capped at 1,500 characters |
| `experiments/memory_sessions.py` | Three sessions on the same lesson, three ways to remember: no memory, Claude's memory tool, or our own store, where the save's last check refuses a repeated quote in code. Each session: the Lesson 4.2 loop, at most 12 turns and $0.10 |

Run it from your `quiz-forge` project, with the environment active:

```bash
python -m experiments.memory_sessions L1.3    # three sessions each: no memory, the memory tool, our own store
cat runs/memory/L1.3/* runs/store/L1.3.json   # Claude's notes, then our store
```

Nine sessions, 54 Claude Haiku 4.5 requests on camera: about 41 cents. On camera, lesson L1.3: no memory 6 different quotes, none repeated this time, $0.1299; the memory tool 6 different, none repeated, $0.1978 (session 1 ran into its 10-cent budget); our own store 6 different, none repeated, $0.0807, about 21,000 input tokens per session. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
