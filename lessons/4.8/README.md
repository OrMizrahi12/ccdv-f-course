# Lesson 4.8: Agent frameworks

The lesson's files, exactly as they were typed on camera (commit `Agent frameworks: what each would add, and a router in our code, Pydantic AI and LangGraph, measured`, on top of Lesson 4.7).

| File | What it is |
|---|---|
| `experiments/framework_weight.py` | What each framework would add to the project, without installing it. For `pydantic-ai-slim[anthropic]`, `langgraph` and `strands-agents[anthropic]`, it runs `pip install --dry-run --quiet --report -` and reads pip's JSON report: every package the install would bring in, and which of the packages pinned in `requirements.txt` it would change. One line per framework |
| `experiments/router.py` | The router pattern three ways, on the same four requests (a quote check, a practice question, a second quote check, an off-topic one), with the same `Route` model (kind `check`, `write` or `other`, plus the lesson id, quote or topic) and the same paths in `act()`. Our own code: `client.messages.parse(..., output_format=Route)`, then an if. Pydantic AI: `Agent(f"anthropic:{cfg.model}", output_type=Route, instructions=ROUTER)` and `run_sync`. LangGraph: a `StateGraph` with a classify node (our own call), `add_conditional_edges` on the route's kind, and one node per path. For each way: the routes, the input tokens, the cost and the time |

Run them from your `quiz-forge` project, with the environment active. The two frameworks go into a separate environment outside the project, so `requirements.txt` doesn't change:

```bash
python -m experiments.framework_weight     # installs nothing
python -m venv ~/.venvs/frameworks         # outside the project
~/.venvs/frameworks/bin/pip install -q anthropic==1.11.0 python-dotenv==1.2.4 "pydantic-ai-slim[anthropic]==2.54.0" langgraph==1.2.14
~/.venvs/frameworks/bin/python -m experiments.router
```

Twelve Claude Haiku 4.5 requests: under a cent. On camera: Pydantic AI would add 5 packages and LangGraph 25, changing none of ours; Strands Agents would add 19 and move `anthropic` 1.11.0 back to 0.125.0, so it isn't installed. All three ways routed check, write, check, other: our code and LangGraph 1,504 input tokens ($0.0020), Pydantic AI 3,140 ($0.0045). The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
