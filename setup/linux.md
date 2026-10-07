# Setup sheet: Linux (Ubuntu and Debian; also WSL on Windows)

This is the system the lesson is filmed on (Ubuntu 24.04, Python 3.12.3). The commands are exactly the ones on screen. Checked against the official install docs on 2026-10-07 (links at the end). On WSL, run everything inside your WSL terminal.

Open a terminal with **Ctrl+Alt+T**.

## 1. VS Code

1. Download the **.deb** for Debian/Ubuntu from https://code.visualstudio.com/download.
2. Install it (Tab completes the file name):
   ```bash
   cd ~/Downloads
   sudo apt install ./code_<version>_amd64.deb
   ```
   Answer **Y** to continue, and **Yes** when the installer asks to add Microsoft's apt repository, so VS Code updates with the rest of the system.
3. Check: `code --version`.
4. In VS Code, open Extensions (**Ctrl+Shift+X**) and install **Python** (publisher Microsoft, verified) and **Claude Code for VS Code** (publisher Anthropic, verified). Watch out for lookalikes.

On WSL, install VS Code on Windows instead (see the Windows sheet) and open your WSL folder from it.

## 2. Python, venv and git

Ubuntu ships Python 3 (the command is `python3`); the SDK needs 3.10 or later. Check: `python3 --version`.

On a fresh Ubuntu, `python3 -m venv .venv` fails because the venv package isn't installed. Install it, and git, once for the machine:

```bash
sudo apt install python3-venv git
```

## 3. The project

```bash
mkdir -p ~/projects/quiz-forge
cd ~/projects/quiz-forge
code .
```

In VS Code, open the terminal with **Ctrl+`**, then:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install anthropic==1.11.0 python-dotenv==1.2.4
pip freeze > requirements.txt
cat requirements.txt
```

The prompt starts with `(.venv)` once the environment is active. Or use `lessons/0.3/requirements.txt` from this repository directly: `pip install -r requirements.txt`.

## 4. git, .env and the first call

```bash
git init -b main
echo 'ANTHROPIC_API_KEY=paste-your-key-here' > .env.example
cp .env.example .env
code .env
```

Paste your API key over the placeholder in `.env` and save. Never paste it into code or chat. Then create `.gitignore` with three lines (or copy `lessons/0.3/.gitignore`):

```
.env
.venv/
__pycache__/
```

Check that git can't see `.env`, add `first_call.py` from `lessons/0.3/`, run it, and commit:

```bash
git status
python first_call.py
git add .
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -m "Set up the quiz-forge workbench"
git log --oneline
```

`git status` should list `.env.example`, `.gitignore` and `requirements.txt` (and `first_call.py`), never `.env` or `.venv`. For a private address, use your GitHub no-reply email (GitHub: Settings, Emails).

## 5. Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
claude --version
```

No admin rights needed; it installs into your home folder (`~/.local/bin/claude`) and updates itself in the background. If the shell says `claude` isn't found, open a new terminal so it picks up the new path.

Start it in the project with `claude`, pick a theme, choose **1. Claude account with subscription** (Pro or Max), and sign in in the browser. Claude Code needs a Pro, Max, Team, Enterprise or Console account; the free plan doesn't include it. Answer **Yes** to the terminal setup, trust the folder (the default answer is **No, exit**: press Down first), then set the course model: `/model sonnet`.

Other ways to install: signed apt, dnf and apk repositories (see the setup page).

## Official sources
- VS Code on Linux: https://code.visualstudio.com/docs/setup/linux
- venv: https://docs.python.org/3/library/venv.html
- Claude Code setup: https://code.claude.com/docs/en/setup
- Python SDK (requires Python 3.10+): https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/python
