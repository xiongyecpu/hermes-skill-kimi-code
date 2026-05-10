# Kimi Code — Hermes Skill

Delegate coding tasks to [Kimi Code CLI](https://www.kimi.com/code/) (Moonshot AI's autonomous coding agent) via the Hermes terminal. Kimi Code CLI can read/edit code, execute shell commands, search/fetch web pages, and autonomously plan and adjust actions.

## How to Trigger This Skill in Hermes

You can invoke this skill using natural language phrases like:

- "Ask Kimi to refactor the auth module"
- "Use Kimi Code to fix the bug in user_service.py"
- "Have Kimi review the changes in this PR"
- "Let Kimi handle the database migration"
- "Run Kimi on this codebase to add unit tests"
- "Start a Kimi session for the frontend cleanup"
- "Background task: Kimi implements the payment feature"
- "Interactive Kimi: debug the API latency issue"

## Installation

### 1. Install Kimi Code CLI

Choose one of the following methods:

**macOS/Linux (curl):**
```bash
curl -LsSf https://code.kimi.com/install.sh | bash
```

**Windows PowerShell:**
```powershell
Invoke-RestMethod https://code.kimi.com/install.ps1 | Invoke-Expression
```

**Using uv (recommended for developers):**
```bash
uv tool install --python 3.13 kimi-cli
```

### 2. Authenticate

Run `kimi`, then enter `/login` to configure your account (OAuth or API Key).

```bash
kimi
> /login
```

### 3. Verify Installation

```bash
kimi --version
```

### 4. Install This Skill

Copy the `kimi-code` skill folder to your Hermes skills directory:

```bash
cp -r skills/kimi-code ~/.hermes/skills/
```

## Quick Start

### One-Shot Task (Print Mode)

Run a single task and get the result:

```
terminal(command="kimi --print -p 'Add error handling to all API calls in src/'", workdir="/path/to/project", timeout=120)
```

### Interactive Session (tmux)

Start a multi-turn session for iterative work:

```
terminal(command="tmux new-session -d -s kimi-work -x 140 -y 40")
terminal(command="tmux send-keys -t kimi-work 'cd /path/to/project && kimi' Enter")
terminal(command="sleep 5 && tmux send-keys -t kimi-work 'Refactor the auth module to use JWT tokens' Enter")
```

### Background Task

Run long tasks in the background:

```
terminal(command="kimi --print -p 'Implement the feature described in plan.md'", workdir="~/project", background=true, timeout=600)
```

## Key Features

- **Print Mode** (`--print`) — Non-interactive, auto-approves all actions
- **Interactive Mode** — Full conversational REPL via tmux
- **PR Reviews** — Automated code review with diff analysis
- **Parallel Work** — Use git worktrees to run multiple tasks simultaneously
- **Session Management** — Resume previous sessions with `--continue`
- **MCP Support** — Extend with external tools and servers
- **Multiple Providers** — Kimi, OpenAI, Anthropic, Gemini, VertexAI

## Documentation

- **[SKILL.md](./SKILL.md)** — Complete orchestration guide for Hermes agents
- **[Kimi Code Official Docs](https://www.kimi.com/code/docs)** — Official documentation
- **[AGENTS.md](https://www.kimi.com/code/docs/project-context)** — Project context format

## Related Skills

- **[claude-code](../claude-code/)** — Claude Code CLI integration
- **[codex](../codex/)** — Codex AI integration
- **[opencode](../opencode/)** — OpenCode integration

## License

Apache License 2.0

## 中文文档

[中文版文档](./README.zh-CN.md)
