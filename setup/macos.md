# Setup sheet: macOS 13 or later

The lesson is filmed on Ubuntu Linux. This sheet gives the same workbench on a Mac, step for step. Every command below was checked against the official install docs on 2026-10-07 (links at the end).

Open **Terminal** (Spotlight: **Cmd+Space**, type `Terminal`).

## 1. VS Code

1. Download VS Code for macOS from https://code.visualstudio.com/download (Universal works on Intel and Apple silicon).
2. Open the downloaded file and drag **Visual Studio Code.app** into **Applications**. Launch it.
3. Add the `code` command: open the Command Palette (**Cmd+Shift+P**), run **Shell Command: Install 'code' command in PATH**, then restart Terminal. Check: `code --version`.
4. In VS Code, open Extensions (**Cmd+Shift+X**) and install **Python** (publisher Microsoft, verified) and **Claude Code for VS Code** (publisher Anthropic, verified). Watch out for lookalikes.

## 2. Python 3.10 or later

The Python SDK needs Python 3.10 or later. The macOS system Python may be older, so install the current one:

1. Download the macOS installer (`.pkg`) from https://www.python.org/downloads/macos/ and double-click it. It includes `pip` and `venv`.
2. Check in a new Terminal window: `python3 --version`.

## 3. git

Either install Apple's Command Line Tools (they include git):

```bash
xcode-select --install
```

or, with Homebrew: `brew install git`. Check: `git --version`.

## 4. The project

```bash
mkdir -p ~/projects/quiz-forge
cd ~/projects/quiz-forge
code .
```

In VS Code, open the terminal with **Ctrl+`** (it opens in the project folder), then:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install anthropic==1.11.0 python-dotenv==1.2.4
pip freeze > requirements.txt
```

The prompt starts with `(.venv)` once the environment is active. Or use `lessons/0.3/requirements.txt` from this repository directly: `pip install -r requirements.txt`.

## 5. git, .env and the first call

```bash
git init -b main
```

Copy the `.gitignore`, `.env.example` and `first_call.py` from `lessons/0.3/` into the folder (in Finder, press **Cmd+Shift+.** to see files that start with a dot). Then:

```bash
cp .env.example .env
code .env
```

Paste your API key over the placeholder in `.env` and save. Never paste it into code or chat. Check that git can't see `.env`:

```bash
git status
```

You should see `.env.example`, `.gitignore`, `first_call.py` and `requirements.txt`, and **not** `.env` or `.venv`. Run the first call and commit:

```bash
python first_call.py
git add .
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -m "Set up the quiz-forge workbench"
git log --oneline
```

For a private address, use your GitHub no-reply email (GitHub: Settings, Emails).

## 6. Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

No administrator rights needed. Open a new Terminal and check: `claude --version`. If the shell says `claude` isn't found, open a new terminal so it picks up the new path, or see Anthropic's "Fix your PATH" troubleshooting page.

Start it in the project with `claude`, pick a theme, choose **1. Claude account with subscription** (Pro or Max), and sign in in the browser. Claude Code needs a Pro, Max, Team, Enterprise or Console account; the free plan doesn't include it. Trust the folder (the default answer is **No, exit**: move down first), then set the course model: `/model sonnet`.

Another way to install: `brew install --cask claude-code` (doesn't update itself; run `brew upgrade claude-code`).

## Official sources
- VS Code on macOS: https://code.visualstudio.com/docs/setup/mac
- Python on macOS: https://docs.python.org/3/using/mac.html
- venv: https://docs.python.org/3/library/venv.html
- git on macOS: https://git-scm.com/install/mac
- Claude Code setup: https://code.claude.com/docs/en/setup
- Python SDK (requires Python 3.10+): https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/python
