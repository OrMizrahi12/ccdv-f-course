# Lesson 1.3: REST, JSON and the Messages API

The lesson's files, exactly as they were typed on camera (commit `Call the Messages API with curl, Python and TypeScript`).

| File | What it is |
|---|---|
| `experiments/request.json` | The request body: `model`, `max_tokens`, `system` and `messages` |
| `experiments/raw_call.sh` | Sends a JSON file to `POST https://api.anthropic.com/v1/messages` with curl and the three headers; saves the headers to `runs/headers.txt` and the body to `runs/reply.json`, and prints the status code |
| `experiments/no_max_tokens.json` | The same request without `max_tokens` (it gets a 400) |
| `experiments/sdk_call.py` | The same request with the Python SDK, then a call to a model that doesn't exist (404, caught as `NotFoundError`) |
| `ts/generate.ts` | The same request with the TypeScript SDK |
| `ts/package.json`, `ts/package-lock.json` | The `ts/` package: `"type": "module"` and `@anthropic-ai/sdk` 0.132.0 |

Run it from your `quiz-forge` project (Lessons 0.3 to 1.2), with the environment active and a `runs/` folder:

```bash
set -a; source .env; set +a                                  # load the key into this shell; prints nothing
bash experiments/raw_call.sh experiments/request.json        # HTTP 200
bash experiments/raw_call.sh experiments/no_max_tokens.json  # HTTP 400: max_tokens: Field required
python experiments/sdk_call.py
cd ts && npm install && cd ..                                # Node.js 20 or later; the lesson used Node 24 from nvm
node --env-file=.env ts/generate.ts
```

Windows PowerShell versions of every step, and the Node.js install for Windows, are in the lesson's student kit. The whole hands-on cost about a tenth of a cent: three small Claude Haiku 4.5 calls. The errors are not billed.
