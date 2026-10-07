# Lesson 0.3: The developer workbench

The `quiz-forge` project exactly as it stands at the end of the lesson (first commit `Set up the quiz-forge workbench`).

| File | What it is |
|---|---|
| `first_call.py` | The first Messages API call: Claude Haiku 4.5, `max_tokens=200`; prints the text, stop reason, token usage and message ID |
| `requirements.txt` | The 16 pinned packages from `pip freeze` (anthropic 1.11.0, python-dotenv 1.2.4 and their dependencies) |
| `.gitignore` | `.env`, `.venv/`, `__pycache__/` |
| `.env.example` | The template. Copy it to `.env` and paste your own key; `.env` is never committed |

Run it:

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows PowerShell: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
cp .env.example .env               # then paste your key into .env
python first_call.py
```

Setup for your operating system: [`setup/`](../../setup/) (Windows, macOS, Linux).
