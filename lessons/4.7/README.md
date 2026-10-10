# Lesson 4.7: Four ways to build an agent

The lesson's files, exactly as they were typed on camera (commit `Four ways to build an agent: our loop, the Tool Runner, the Agent SDK and Managed Agents, one job, measured`, on top of Lesson 4.6).

| File | What it is |
|---|---|
| `requirements.txt` | The course's pinned packages, now with the Agent SDK (`claude-agent-sdk==0.2.165`) and what it brings: 18 new lines, written by `pip freeze` |
| `experiments/four_ways.py` | One small job, four ways to build the agent: our own loop, the Tool Runner, the Agent SDK (Claude Code's loop and tools, in our process) and Managed Agents (Anthropic's harness and sandbox, the lesson uploaded as a file, a 10-cent session budget, everything deleted afterwards). One row each: tool calls, cost, seconds, and the right quote or not |

Run them from your `quiz-forge` project, with the environment active (the Agent SDK brings its own Claude Code binary; Managed Agents is in beta, and on by default for API accounts):

```bash
pip install -r requirements.txt     # the Agent SDK, claude-agent-sdk 0.2.165, and what it brings
python -m experiments.four_ways     # one job: our loop, the Tool Runner, the Agent SDK, Managed Agents
```

About ten Claude Haiku 4.5 requests: about 5 cents. On camera: our loop and the Tool Runner 1 tool call each, under a cent, under four seconds; the Agent SDK 2 tool calls, $0.0227, 15.4 s; Managed Agents 3 tool calls, $0.0183, 79.5 s in Anthropic's sandbox (list cost 2 cents); all four found the right quote. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
