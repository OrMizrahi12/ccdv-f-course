# Setup sheet: Windows 10 or 11

The lesson is filmed on Ubuntu Linux. This sheet gives the same workbench on Windows, step for step. Every command below was checked against the official install docs on 2026-10-07 (links at the end).

Use **PowerShell** for the commands (Start menu, type `PowerShell`). Your prompt starts with `PS C:\Users\YourName>`.

## 1. VS Code

1. Download the **User Installer** for Windows from https://code.visualstudio.com/download and run it. The User setup needs no administrator rights.
2. Keep the default option that adds VS Code to your `PATH`.
3. Open a **new** PowerShell window and check: `code --version`.
4. In VS Code, open Extensions (**Ctrl+Shift+X**) and install **Python** (publisher Microsoft, verified) and **Claude Code for VS Code** (publisher Anthropic, verified). Watch out for lookalikes.

## 2. Python 3.10 or later

The Python SDK needs Python 3.10 or later.

1. Install the **Python Install Manager**: from the Microsoft Store (search "Python"), or from https://www.python.org/downloads/.
2. Open a new PowerShell window and check: `python --version`. (`py` works too and lets you pick a version.)

## 3. git

1. Install **Git for Windows** from https://git-scm.com/install/windows, or run:
   ```powershell
   winget install --id Git.Git -e --source winget
   ```
2. Open a new PowerShell window and check: `git --version`.

Git for Windows also gives Claude Code its Bash tool. Without it, Claude Code uses PowerShell instead.

## 4. The project

```powershell
mkdir $HOME\projects\quiz-forge
cd $HOME\projects\quiz-forge
code .
```

In VS Code, open the terminal with **Ctrl+`** (it opens in the project folder), then:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

If PowerShell refuses to run `Activate.ps1`, allow local scripts once for your account, then activate again:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

In **cmd.exe** instead of PowerShell, activate with `.venv\Scripts\activate.bat`.

The prompt now starts with `(.venv)`. Install the pinned packages and record them:

```powershell
pip install anthropic==1.11.0 python-dotenv==1.2.4
pip freeze > requirements.txt
```

Or use `lessons/0.3/requirements.txt` from this repository directly: `pip install -r requirements.txt`.

## 5. git, .env and the first call

```powershell
git init -b main
```

Copy the `.gitignore`, `.env.example` and `first_call.py` from `lessons/0.3/` into the folder. Then:

```powershell
copy .env.example .env
code .env
```

Paste your API key over the placeholder in `.env` and save. Never paste it into code or chat. Check that git can't see `.env`:

```powershell
git status
```

You should see `.env.example`, `.gitignore`, `first_call.py` and `requirements.txt`, and **not** `.env` or `.venv`. Run the first call:

```powershell
python first_call.py
```

Then commit:

```powershell
git add .
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -m "Set up the quiz-forge workbench"
git log --oneline
```

For a private address, use your GitHub no-reply email (GitHub: Settings, Emails).

## 6. Claude Code

In PowerShell (no administrator rights needed):

```powershell
irm https://claude.ai/install.ps1 | iex
```

In cmd.exe the command is:

```bat
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

Open a **new** terminal and check: `claude --version`. If `claude` isn't recognized, the install folder isn't on your `PATH` yet: open a new terminal, or see Anthropic's "Fix your PATH" troubleshooting page.

Start it in the project with `claude`, pick a theme, choose **1. Claude account with subscription** (Pro or Max), and sign in in the browser. Claude Code needs a Pro, Max, Team, Enterprise or Console account; the free plan doesn't include it. Trust the folder (the default answer is **No, exit**: move down first), then set the course model: `/model sonnet`.

Other ways to install: `winget install Anthropic.ClaudeCode` (doesn't update itself; run `winget upgrade Anthropic.ClaudeCode`), or install inside WSL with the Linux sheet.

## Official sources
- VS Code on Windows: https://code.visualstudio.com/docs/setup/windows
- Python on Windows: https://docs.python.org/3/using/windows.html
- venv activation and the execution policy: https://docs.python.org/3/library/venv.html
- Git for Windows: https://git-scm.com/install/windows
- Claude Code setup: https://code.claude.com/docs/en/setup
- Python SDK (requires Python 3.10+): https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/python
