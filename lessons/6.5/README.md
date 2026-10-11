# Lesson 6.5: Connecting enterprise systems

The code from the lesson, as it was committed on camera (`b116bf3`, `30b89a6` and `6f823ad`).

| File | What it is |
|---|---|
| `mcp_server/auth.py` | `HashedTokenVerifier`: a development-only token check against a file of SHA-256 hashes kept outside the repo. In production you verify a JWT from your identity provider instead. |
| `mcp_server/server.py` | The library server with `token_verifier` and `AuthSettings` (a placeholder issuer, this server as the resource, the `library:read` scope). Over stdio nothing changes. |
| `scripts/mint_dev_token.py` | Writes a new token to `~/.config/quiz-forge/mcp-token` (mode 600) and only its hash to `mcp-token-hashes`. It never prints the token. |
| `tests/test_mcp_auth.py` | A registered token passes; any other is refused. |
| `scripts/auth_probe.sh` | Knocks four ways: no token (401), the metadata, with the token (200), from another Host name (421). It never prints the token. |
| `scripts/mcp_headers.sh` | Claude Code's `headersHelper`: prints the Authorization header when Claude Code connects. Don't run it by hand: its output is a secret. |
| `.claude/settings.json` | The shared project settings, with the library's three read tools in `allow`. |
| `docs/security-review.md` | A starting frame for a security review (an invented regulated bank; not legal advice). |

Start from your `quiz-forge` project at the Lesson 6.4 state, with port 8765 free. Copy the files in, then:

```bash
.venv/bin/python -m scripts.mint_dev_token
.venv/bin/python -m pytest -q
.venv/bin/python -m mcp_server.server --http > runs/http.log 2>&1 &
scripts/auth_probe.sh
claude mcp add-json quiz-library-http '{"type": "http", "url": "http://127.0.0.1:8765/mcp", "headersHelper": "scripts/mcp_headers.sh"}'
```

Cost: no API calls in this lesson; Claude Code runs on your own plan. `https://idp.example.com` is a placeholder: no identity provider runs here. The student kit has the full steps for Windows, macOS and Linux, including the token rotation and the permission rules.
