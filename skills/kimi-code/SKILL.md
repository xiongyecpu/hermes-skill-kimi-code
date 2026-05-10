---
name: kimi-code
description: "Delegate coding tasks to Kimi Code CLI — Moonshot AI's autonomous coding agent. Supports print mode, interactive sessions, PR reviews, parallel work, and advanced customization."
version: 2.0.0
author: xiongye
license: Apache-2.0
metadata:
  hermes:
    tags: [Coding-Agent, Kimi, Moonshot-AI, Code-Review, Refactoring, PTY, Automation]
    related_skills: [claude-code, codex, opencode, hermes-agent]
---

# Kimi Code — Hermes Orchestration Guide

Delegate coding tasks to [Kimi Code CLI](https://www.kimi.com/code/) (Moonshot AI's autonomous coding agent) via the Hermes terminal. Kimi Code CLI can read/edit code, execute shell commands, search/fetch web pages, and autonomously plan and adjust actions.

## Prerequisites

- **Install** (choose one):
  - `curl -LsSf https://code.kimi.com/install.sh | bash` (macOS/Linux)
  - `Invoke-RestMethod https://code.kimi.com/install.ps1 | Invoke-Expression` (Windows PowerShell)
  - `uv tool install --python 3.13 kimi-cli` (if you already have [uv](https://docs.astral.sh/uv/))
- **Auth**: run `kimi`, then enter `/login` to configure platform and API key (OAuth or API Key)
- **Verify**: `kimi --version`
- **Update**: `uv tool upgrade kimi-cli --no-cache`
- **Uninstall**: `uv tool uninstall kimi-cli`

## Two Orchestration Modes

Hermes interacts with Kimi Code in two fundamentally different ways. Choose based on the task.

### Mode 1: Print Mode (`--print`) — Non-Interactive (PREFERRED for most tasks)

Print mode runs a one-shot task, returns the result, and exits. No PTY needed. No interactive prompts. Auto-approves all actions (implicitly enables `--yolo`).

```
# Simple task
terminal(command="kimi --print -p 'Add error handling to all API calls in src/'", workdir="/path/to/project", timeout=120)

# Quiet mode — only final answer, no intermediate steps
terminal(command="kimi --quiet -p 'Explain what this project does'", workdir="/path/to/project")

# Background — long coding task
terminal(command="kimi --print -p 'Refactor the auth module to use JWT tokens'", workdir="/path/to/project", background=true, timeout=600)
```

**When to use print mode:**
- One-shot coding tasks (fix a bug, add a feature, refactor)
- CI/CD automation and scripting
- Structured data extraction with `--output-format stream-json`
- Piped input processing (`echo '{...}' | kimi --print --input-format=stream-json`)
- Any task where you don't need multi-turn conversation

**Print mode skips ALL interactive dialogs** — no workspace trust prompt, no permission confirmations. This makes it ideal for automation.

### Mode 2: Interactive PTY via tmux — Multi-Turn Sessions

Interactive mode gives you a full conversational REPL where you can send follow-up prompts, use slash commands, and watch Kimi work in real time. **Requires tmux orchestration.**

```
# Start a tmux session
terminal(command="tmux new-session -d -s kimi-work -x 140 -y 40")

# Launch Kimi Code inside it
terminal(command="tmux send-keys -t kimi-work 'cd /path/to/project && kimi' Enter")

# Wait for startup (~3-5 seconds), then send your task
terminal(command="sleep 5 && tmux send-keys -t kimi-work 'Refactor the auth module to use JWT tokens' Enter")

# Monitor progress by capturing the pane
terminal(command="sleep 15 && tmux capture-pane -t kimi-work -p -S -50")

# Send follow-up tasks
terminal(command="tmux send-keys -t kimi-work 'Now add unit tests for the new JWT code' Enter")

# Exit when done
terminal(command="tmux send-keys -t kimi-work '/exit' Enter")
```

**When to use interactive mode:**
- Multi-turn iterative work (refactor → review → fix → test cycle)
- Tasks requiring human-in-the-loop decisions
- Exploratory coding sessions
- When you need to use Kimi's slash commands (`/compact`, `/plan`, `/model`)

## PTY Dialog Handling (CRITICAL for Interactive Mode)

Kimi Code presents a confirmation dialog on first launch. You MUST handle these via tmux send-keys:

### Dialog: Workspace Trust (first visit to a directory)
```
❯ 1. Yes, I trust this folder    ← DEFAULT (just press Enter)
  2. No, exit
```
**Handling:** `tmux send-keys -t <session> Enter` — default selection is correct.

### Robust Dialog Handling Pattern
```
# Launch Kimi Code
terminal(command="tmux send-keys -t kimi-work 'cd /path/to/project && kimi' Enter")

# Handle trust dialog (Enter for default "Yes")
terminal(command="sleep 4 && tmux send-keys -t kimi-work Enter")

# Now wait for Kimi to work
terminal(command="sleep 15 && tmux capture-pane -t kimi-work -p -S -60")
```

**Note:** After the first trust acceptance for a directory, the trust dialog won't appear again.

## CLI Subcommands

| Subcommand | Description |
|------------|-------------|
| `kimi` | Start interactive session |
| `kimi login` | Login to Kimi account (OAuth) |
| `kimi logout` | Logout and clear credentials |
| `kimi info` | Show version and protocol info |
| `kimi acp` | Start ACP server (for IDE integration) |
| `kimi mcp` | Manage MCP server configurations |
| `kimi term` | Run TUI terminal interface |
| `kimi vis` | Run agent tracing visualizer (tech preview) |
| `kimi web` | Start web UI server |
| `kimi export` | Export session data as ZIP |

## Print Mode Deep Dive

### Structured JSON Output
```
terminal(command="kimi --print -p 'Analyze auth.py for security issues' --output-format stream-json", workdir="/project", timeout=120)
```

Returns JSONL (newline-delimited JSON) output:
```jsonl
{"role":"assistant","content":"The analysis found..."}
{"role":"tool","tool_call_id":"tc_1","content":"Tool execution result"}
```

### Piped Input
```
# Pipe a file for analysis
terminal(command="cat src/auth.py | kimi --print -p 'Review this code for bugs' --max-steps-per-turn 10", timeout=60)

# Pipe command output
terminal(command="git diff HEAD~3 | kimi --print -p 'Summarize these changes'", timeout=60)
```

### Exit Codes (for scripting/CI)

| Exit code | Meaning |
|-----------|---------|
| `0` | Success — task completed |
| `1` | Permanent failure — config error, auth failure, quota exhausted |
| `2` | Transient failure — rate limit (429), server error (5xx), timeout (safe to retry) |
| `3` | Interrupted — user interrupted with Ctrl-C |

### Ralph Loop Mode

[Ralph](https://ghuntley.com/ralph/) is a technique that puts the agent in a loop: the same prompt is repeatedly fed to the agent, letting it iterate on a task continuously.

```
terminal(command="kimi --print -p 'Keep improving the test coverage until it reaches 90%' --max-ralph-iterations 10", workdir="~/project", timeout=300)
```

The agent loops until it outputs `<choice>STOP</choice>` or reaches the iteration limit.

## Complete CLI Flags Reference

### Session & Environment
| Flag | Shorthand | Effect |
|------|-----------|--------|
| `--print` | | Non-interactive mode, auto-approves all actions, outputs to stdout |
| `--quiet` | | Shorthand for `--print --output-format text --final-message-only` |
| `--prompt TEXT` | `-p`, `-c` | Task prompt (non-interactive) |
| `--continue` | `-C` | Continue most recent session in current directory |
| `--session [ID]` / `--resume [ID]` | `-S` / `-r` | Resume session. With ID: resume specific session; without ID: interactive picker |
| `--work-dir PATH` | `-w` | Working directory (default: current) |
| `--add-dir PATH` | | Add extra directory to workspace scope (repeatable) |
| `--config STRING` | | Load TOML/JSON configuration string |
| `--config-file PATH` | | Load configuration file (default: `~/.kimi/config.toml`) |

### Model & Performance
| Flag | Shorthand | Effect |
|------|-----------|--------|
| `--model NAME` | `-m` | Specify LLM model (default from config) |
| `--thinking` | | Enable thinking mode (deeper reasoning) |
| `--no-thinking` | | Disable thinking mode |
| `--plan` | | Start in plan mode (read-only planning) |
| `--max-steps-per-turn N` | | Override max steps per turn (default: 100) |
| `--max-retries-per-step N` | | Override max retries per step (default: 3) |
| `--max-ralph-iterations N` | | Ralph loop iterations. `0`=off, `-1`=infinite |

### Permission & Safety
| Flag | Shorthand | Effect |
|------|-----------|--------|
| `--yolo` | `-y` | Auto-approve all operations |
| `--yes` | | Alias for `--yolo` |
| `--auto-approve` | | Alias for `--yolo` |

### Output & Input Format
| Flag | Effect |
|------|--------|
| `--output-format FORMAT` | `text` (default) or `stream-json` |
| `--input-format FORMAT` | `text` (default) or `stream-json` (stdin) |
| `--final-message-only` | Only output the final assistant message |
| `--verbose` | Verbose output |
| `--debug` | Debug logging to `~/.kimi/logs/kimi.log` |

### Agent & MCP
| Flag | Effect |
|------|--------|
| `--agent NAME` | Built-in agent: `default`, `okabe` |
| `--agent-file PATH` | Custom agent file |
| `--skills-dir PATH` | Add extra skills directory (repeatable) |
| `--mcp-config-file PATH` | Load MCP config file (repeatable) |
| `--mcp-config JSON` | Load MCP config JSON string (repeatable) |

## Session Management

```
# Continue most recent session in current directory
terminal(command="kimi --continue --print -p 'Continue where we left off'", workdir="~/project")

# Resume specific session by ID
terminal(command="kimi --session abc123 --print -p 'Pick up this session'", workdir="~/project")

# Interactive session picker (tmux only)
terminal(command="kimi --session", workdir="~/project", pty=true)
```

Session state is automatically persisted and restored:
- Approval decisions (YOLO mode, per-operation approvals)
- Plan mode state
- Subagent instances
- Extra directories added via `--add-dir`

## Model and Thinking Control

```
# Use a specific model
terminal(command="kimi --print -m kimi-for-coding -p 'Solve this hard problem'", workdir="~/project")

# Enable thinking mode (deeper reasoning)
terminal(command="kimi --thinking --print -p 'Analyze the architecture tradeoffs'", workdir="~/project")

# Disable thinking mode (faster, cheaper)
terminal(command="kimi --no-thinking --print -p 'Quick formatting fix'", workdir="~/project")
```

**Note**: `kimi-for-coding` is the fixed model ID. The backend automatically maps it to the latest released model — you don't need to change configuration for model upgrades.

## PR Reviews

### Quick Review (Print Mode)
```
terminal(command="cd /path/to/repo && git diff main...feature-branch | kimi --print -p 'Review this diff for bugs, security issues, and style problems. Be thorough.'", timeout=60)
```

### Deep Review (Interactive + Worktree)
```
terminal(command="git worktree add /tmp/pr-review pr-branch", workdir="~/project")
terminal(command="tmux new-session -d -s review -x 140 -y 40")
terminal(command="tmux send-keys -t review 'cd /tmp/pr-review && kimi' Enter")
terminal(command="sleep 5 && tmux send-keys -t review 'Review all changes vs main. Check for bugs, security issues, race conditions, and missing tests.' Enter")
terminal(command="sleep 30 && tmux capture-pane -t review -p -S -60")
```

## Parallel Work with Worktrees

```
# Create worktrees
terminal(command="git worktree add -b fix/issue-78 /tmp/issue-78 main", workdir="~/project")
terminal(command="git worktree add -b fix/issue-99 /tmp/issue-99 main", workdir="~/project")

# Launch Kimi in each (background)
terminal(command="kimi --print -p 'Fix issue #78: <description>. Commit when done.'", workdir="/tmp/issue-78", background=true)
terminal(command="kimi --print -p 'Fix issue #99: <description>. Commit when done.'", workdir="/tmp/issue-99", background=true)

# Monitor all
process(action="list")

# After completion, push and create PRs
terminal(command="cd /tmp/issue-78 && git push -u origin fix/issue-78")
terminal(command="gh pr create --repo user/repo --head fix/issue-78 --title 'fix: ...' --body '...'")

# Cleanup
terminal(command="git worktree remove /tmp/issue-78", workdir="~/project")
```

## Background Mode (Long Tasks)

```
# Start in background with print mode
terminal(command="kimi --print -p 'Implement the feature described in plan.md'", workdir="~/project", background=true, timeout=600)
# Returns session_id

# Monitor progress
process(action="poll", session_id="<id>")
process(action="log", session_id="<id>")

# Kill if needed
process(action="kill", session_id="<id>")
```

## Configuration

Default config: `~/.kimi/config.toml` (also supports JSON)

Key settings:
```toml
# Model settings
default_model = "kimi-for-coding"
default_thinking = false

# Behavior
default_yolo = false
default_plan_mode = false

# Loop control
[loop_control]
max_steps_per_turn = 100
max_retries_per_step = 3

# Background tasks
[background]
max_concurrent = 4
```

Kimi Code CLI supports multiple providers: Kimi Code, OpenAI, Anthropic, Gemini, VertexAI. See [official providers docs](https://www.kimi.com/code/docs/kimi-code-cli/configuration/providers-and-models.html).

**Important**: Kimi Code (`api.kimi.com`) and Kimi Open Platform (`api.moonshot.cn`) are completely separate account systems. API keys are NOT interchangeable.

## Context Management

- **Auto-compaction**: Kimi CLI auto-compacts context when usage reaches ~85%
- **Manual compression**: Use `/compact [focus]` to summarize and compress context
- **Clear context**: Use `/clear` to wipe conversation history
- **Context size**: 262K tokens (model-dependent)
- **Status bar**: Shows `context: 42.0% (4.2k/10.0k)` format

## AGENTS.md — Project Context File

Kimi Code CLI auto-loads `AGENTS.md` from the project root. Use it to persist project context:

```markdown
# Project: My API

## Architecture
- FastAPI backend with SQLAlchemy ORM
- PostgreSQL database, Redis cache

## Key Commands
- `make test` — run full test suite
- `make lint` — ruff + mypy

## Code Standards
- Type hints on all public functions
- 4-space indentation for Python
```

Generate one automatically: run `/init` inside Kimi CLI.

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Shell` | Execute shell commands |
| `ReadFile` | Read file contents |
| `WriteFile` | Write/create files |
| `StrReplaceFile` | Find and replace in files |
| `Glob` | File pattern matching |
| `Grep` | Search file contents |
| `ReadMediaFile` | Read image/video files |
| `SearchWeb` | Web search |
| `FetchURL` | Fetch web page content |
| `Agent` | Spawn subagents (coder, explore, plan) |
| `AskUserQuestion` | Ask user for structured input |
| `SetTodoList` | Manage todo list |
| `EnterPlanMode` / `ExitPlanMode` | Toggle plan mode |
| `TaskList` / `TaskOutput` / `TaskStop` | Manage background tasks |

## Built-in Subagents

| Type | Purpose | Available Tools |
|------|---------|-----------------|
| `coder` | General software engineering | Shell, ReadFile, Glob, Grep, WriteFile, StrReplaceFile, SearchWeb, FetchURL |
| `explore` | Read-only codebase exploration | Shell, ReadFile, Glob, Grep, SearchWeb, FetchURL |
| `plan` | Implementation planning | ReadFile, Glob, Grep, SearchWeb, FetchURL |

## Interactive Session: Slash Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show all commands |
| `/compact [focus]` | Compress context to save tokens |
| `/clear` | Wipe conversation history |
| `/plan` | Enter plan mode |
| `/model` | Switch model |
| `/exit` | End session |

## Interactive Session: Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Ctrl+C` | Cancel current input or generation |
| `Ctrl+D` | Exit session |
| `Shift+Tab` | Cycle permission modes |
| `\` + `Enter` | Quick newline |
| `Shift+Enter` | Newline |

## Pitfalls & Gotchas

1. **Command name varies by install method** — may be `kimi` or `kimi-cli`. Check with `which kimi 2>/dev/null || which kimi-cli`. This skill uses `kimi` as default.

2. **No git repo required** — Unlike Codex, Kimi CLI works in ANY directory. No need to `git init` for scratch work.

3. **Print mode is the safest non-interactive option** — Avoid PTY + `background=true`. Prompts may not get submitted correctly. Use `--print` mode instead.

4. **Exit code 2 means retry** — Transient errors (rate limits, server errors, timeouts). Implement retry logic:
   ```bash
   kimi --print -p "Run task"
   code=$?
   if [ $code -eq 2 ]; then
     sleep 10
     kimi --print -p "Run task"
   fi
   ```

5. **Context auto-compaction** — At ~85% usage, context is automatically compressed. For very long sessions, use `/compact` or start a new session with `/new`.

6. **File truncation on multi-file generation** — Like other coding agents, Kimi may silently truncate files when generating many files at once. Always verify after bulk generation:
   ```bash
   python3 -c "import sys; sys.path.insert(0, 'src'); from pkg import module_a, module_b; print('OK')"
   ```

7. **macOS Gatekeeper delay** — First run of `kimi` may take a while due to macOS security checks. Add your terminal to "System Settings → Privacy & Security → Developer Tools" to speed up subsequent launches.

8. **API Key platform mismatch** — `api.kimi.com` (Kimi Code) and `api.moonshot.cn` (Kimi Open Platform) are separate systems. Keys are NOT interchangeable. Use the correct Base URL for your key:
   - Kimi Code OpenAI-compatible: `https://api.kimi.com/coding/v1`
   - Kimi Code Anthropic-compatible: `https://api.kimi.com/coding/`
   - Kimi Open Platform: `https://api.moonshot.cn/v1`

9. **Thinking mode needs model support** — Not all models support thinking. If unsupported, the flag is silently ignored.

10. **Session resumption requires same directory** — `--continue` finds the most recent session for the current working directory.

## Rules for Hermes Agents

1. **Prefer print mode (`--print`) for single tasks** — cleaner, no dialog handling, structured output possible
2. **Use tmux for multi-turn interactive work** — the only reliable way to orchestrate the TUI
3. **Always set `workdir`** — keep Kimi focused on the right project directory
4. **Use `--quiet` for simple queries** — only the final answer, no intermediate noise
5. **Use `background=true` for long tasks** — and monitor with `process` tool
6. **Monitor tmux sessions** — use `tmux capture-pane -t <session> -p -S -50` to check progress
7. **Look for the `>` prompt** — indicates Kimi is waiting for input (done or asking a question)
8. **Clean up tmux sessions** — kill them when done to avoid resource leaks
9. **Check exit codes** — 0=success, 1=permanent fail, 2=retry, 3=interrupted
10. **No git repo needed** — works in any directory, unlike Codex
11. **Verify multi-file output** — run import checks after bulk file generation
12. **Use `--max-steps-per-turn`** to prevent runaway loops on complex tasks
