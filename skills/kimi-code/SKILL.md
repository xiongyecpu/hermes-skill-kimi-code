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

This skill enables Hermes agents to delegate coding tasks to [Kimi Code CLI](https://code.kimi.com), a command-line AI coding assistant by Moonshot AI. Kimi provides competitive code generation, refactoring, and exploration capabilities with support for interactive sessions, background tasks, and worktree-based parallel work.

## Prerequisites

### Installation

Choose one of the following installation methods:

**Via install script (recommended):**
```bash
curl -LsSf https://code.kimi.com/install.sh | bash
```

**Via uv (Python 3.13+ required):**
```bash
uv tool install --python 3.13 kimi-cli
```

### Authentication

After installation, authenticate by running the CLI and using the built-in login command:
```bash
kimi
/login
```

Follow the browser-based authentication flow.

### Verify Installation

```bash
kimi --version
```

### Update

```bash
uv tool upgrade kimi-cli --no-cache
```

### Uninstall

```bash
uv tool uninstall kimi-cli
```

## Two Orchestration Modes

Hermes agents can interact with Kimi Code in two primary modes:

### Mode 1: Print Mode (Preferred)

**Best for:** One-off tasks, automated workflows, CI/CD integration

```bash
kimi --print -p "Implement a binary search tree"
```

- Non-interactive, no PTY required
- Auto-approves all operations (`--yolo` implied)
- Outputs structured JSON or plain text
- Suitable for piping and programmatic consumption
- Use `--quiet` for minimal output (`--print --output-format text --final-message-only`)

### Mode 2: Interactive PTY via tmux

**Best for:** Complex multi-step tasks, debugging, exploratory sessions

```bash
tmux new-session -d -s kimi-work "kimi"
tmux send-keys -t kimi-work "Implement a REST API with error handling" C-m
```

- Full interactive session with confirmation dialogs
- Requires PTY management (tmux recommended)
- Allows observation of agent progress
- Supports `/plan` mode and visual feedback
- Use for long-running tasks where oversight is valuable

## PTY Dialog Handling

When using interactive mode, certain operations trigger confirmation dialogs:

**Trust dialogs** appear for:
- File operations outside work directory
- Shell command execution
- External tool usage

**Handling strategies:**

1. **Automated approval:** Include `--yolo` or `--yes` flags
2. **Pre-approval:** Use `kimi acp` (approve common paths) beforehand
3. **tmux automation:** Send confirmations via `tmux send-keys`
4. **Print mode:** Avoid dialogs entirely

```bash
# Pre-approve common paths
kimi acp

# Auto-approve in session
kimi --yolo -p "Refactor the auth module"

# Send 'y' via tmux
tmux send-keys -t kimi-work "y" C-m
```

## CLI Subcommands

| Subcommand | Description |
|------------|-------------|
| `kimi` | Start interactive coding session |
| `kimi login` | Authenticate with Kimi Code |
| `kimi logout` | Clear authentication credentials |
| `kimi info` | Display version and configuration info |
| `kimi acp` | Approve common paths for file operations |
| `kimi mcp` | Manage MCP server connections |
| `kimi term` | Terminal/UI mode (default interactive) |
| `kimi vis` | Visualization mode (graphs, diagrams) |
| `kimi web` | Launch web UI |
| `kimi export` | Export conversation history |

## Print Mode Deep Dive

Print mode is the primary integration point for Hermes agents.

### Structured JSON Output

```bash
kimi --print --output-format stream-json -p "Add input validation"
```

Returns streaming JSON objects:
```json
{"type": "start", "timestamp": "..."}
{"type": "thinking", "content": "..."}
{"type": "tool_use", "tool": "ReadFile", "input": {"path": "src/main.py"}}
{"type": "tool_result", "tool": "ReadFile", "output": {"content": "..."}}
{"type": "message", "role": "assistant", "content": "I've added validation..."}
{"type": "end", "exit_code": 0}
```

### Piped Input

```bash
echo "Extract the user service into a separate module" | kimi --print
cat task.txt | kimi --print --output-format text
```

### Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | Proceed |
| 1 | Permanent failure | Abort, do not retry |
| 2 | Transient failure | Retry with backoff |
| 3 | Interrupted | User cancelled |

### Ralph Loop Mode

The "Ralph Loop" is Kimi's internal iteration mechanism where the agent:
1. Reads current state
2. Plans next action
3. Executes tool call
4. Observes result
5. Repeats until done

Control via flags:
- `--max-steps-per-turn N` (default: 100)
- `--max-retries-per-step N` (default: 3)
- `--max-ralph-iterations N` (overall loop limit)

## Complete CLI Flags Reference

### Session & Environment

| Flag | Short | Description |
|------|-------|-------------|
| `--continue` | `-C` | Continue last session in current directory |
| `--session [ID]` | `-S` | Attach to specific session ID |
| `--resume [ID]` | `-r` | Resume previous session |
| `--work-dir PATH` | `-w` | Set working directory |
| `--add-dir PATH` | | Add directory to context |
| `--config STRING` | | Named configuration profile |
| `--config-file PATH` | | Use specific config file |

### Model & Performance

| Flag | Short | Description |
|------|-------|-------------|
| `--model NAME` | `-m` | Select model (e.g., `moonshot-v1-128k`, `claude-3-7-sonnet`) |
| `--thinking` | | Enable thinking mode (if supported) |
| `--no-thinking` | | Disable thinking mode |
| `--plan` | | Start in planning mode |
| `--max-steps-per-turn N` | | Max Ralph Loop iterations per turn |
| `--max-retries-per-step N` | | Max retries per tool call |
| `--max-ralph-iterations N` | | Overall loop limit |

### Permission & Safety

| Flag | Short | Description |
|------|-------|-------------|
| `--yolo` | `-y` | Auto-approve all operations |
| `--yes` | | Auto-approve operations |
| `--auto-approve` | | Alias for `--yes` |

### Output & Input

| Flag | Short | Description |
|------|-------|-------------|
| `--output-format FORMAT` | | `text` or `stream-json` |
| `--input-format FORMAT` | | Input format |
| `--final-message-only` | | Output only final response |
| `--quiet` | | Minimal output (implies `--print --output-format text --final-message-only`) |
| `--verbose` | `-v` | Detailed output |
| `--debug` | | Debug-level output |
| `--print` | | Non-interactive print mode |

### Agent & MCP

| Flag | Short | Description |
|------|-------|-------------|
| `--agent NAME` | | Select subagent (coder, explore, plan) |
| `--agent-file PATH` | | Custom agent definition |
| `--skills-dir PATH` | | Skills directory path |
| `--mcp-config-file PATH` | | MCP configuration file |
| `--mcp-config JSON` | | Inline MCP configuration |

## Session Management

### Continuing Sessions

```bash
# Continue last session in current directory
kimi --continue

# Equivalent short form
kimi -C

# Continue with prompt
kimi -C -p "Now add unit tests"
```

### Session Resumption

```bash
# Resume specific session by ID
kimi --session abc123

# Resume from any directory
kimi --work-dir /path/to/project --resume abc123

# Short form
kimi -S abc123
kimi -r abc123
```

### Session Picker

Run `kimi` without arguments in a project directory to see available sessions:
```
Recent sessions in this directory:
  [1] abc123 - "Add authentication" (2 hours ago)
  [2] def456 - "Refactor database" (yesterday)
```

## Model and Thinking Control

### Model Selection

```bash
# Use specific model
kimi --model moonshot-v1-128k -p "Task"

# List available models
kimi info
```

**Common models:**
- `moonshot-v1-128k` - Default Kimi model
- `claude-3-7-sonnet` - Claude 3.7 Sonnet via Kimi
- `claude-3-5-sonnet` - Claude 3.5 Sonnet via Kimi

### Thinking Mode

Enable extended reasoning (model-dependent):

```bash
# Enable thinking
kimi --thinking -p "Complex algorithm optimization"

# Disable explicitly
kimi --no-thinking -p "Simple refactor"
```

**Note:** Not all models support thinking mode. Check model compatibility first.

## PR Reviews

### Quick Review (Print Mode)

```bash
kimi --print -p "Review PR #123: check for security issues"
```

- Fast, non-interactive
- Returns review as text/JSON
- Suitable for automated checks

### Deep Review (Interactive + Worktree)

```bash
# Create worktree for isolated review
git worktree add ../review-123 pr-123
cd ../review-123

# Start interactive review session
kimi --plan
```

- Full context with files
- Can make edits directly
- Preserves original branch

## Parallel Work with Worktrees

Worktree isolation enables parallel task execution:

```bash
# Create multiple worktrees
git worktree add ../feature-a feature-a
git worktree add ../feature-b feature-b

# Run parallel Kimi sessions
kimi --work-dir ../feature-a --print -p "Implement feature A" &
kimi --work-dir ../feature-b --print -p "Implement feature B" &

# Wait for completion
wait
```

**Benefits:**
- True parallelism
- Isolated file systems
- Independent git histories
- No blocking between tasks

## Background Mode

For long-running tasks with process monitoring:

```bash
# Start background task
kimi --print -p "Run full test suite and fix failures" > task.log 2>&1 &

# Monitor progress
tail -f task.log

# Check if still running
ps aux | grep kimi
```

**Configurable via `~/.kimi/config.toml`:**
```toml
[background]
max_concurrent = 4  # Max parallel background tasks
```

## Configuration

### Config File Location

Primary configuration: `~/.kimi/config.toml`

Alternative formats supported: `.json`, `.yaml`

### Key Settings

```toml
[model]
default_model = "moonshot-v1-128k"
default_thinking = false
default_yolo = false

[plan]
default_plan_mode = false

[loop_control]
max_steps_per_turn = 100
max_retries_per_step = 3
max_ralph_iterations = 1000

[background]
max_concurrent = 4

[paths]
approved_dirs = ["/usr/local", "./src"]
```

### Provider Configuration: API Platform Warning

**Important:** Kimi Code (`api.kimi.com`) and Kimi Open Platform (`api.moonshot.cn`) are **separate services with non-interchangeable API keys**.

| Platform | Base URL (OpenAI) | Base URL (Anthropic) | Purpose |
|----------|-------------------|----------------------|---------|
| Kimi Code | `https://api.kimi.com/coding/v1` | `https://api.kimi.com/coding/` | AI coding assistant |
| Kimi Open Platform | `https://api.moonshot.cn/v1` | N/A | General API access |

**Common pitfall:** Using Open Platform key for Code CLI (or vice versa) causes authentication failures. Ensure you're using the correct key for the intended service.

## Context Management

### Auto-Compaction

Kimi automatically compacts context at approximately 85% of token limit (~262K tokens). Older messages are summarized while preserving critical information.

### Manual Context Control

```bash
# Compact context with focus
/compact "Focus on authentication changes"

# Clear entire context
/clear
```

### Context Size Indicators

In interactive mode, context usage appears in prompts:
```
[Context: 223K/262K tokens]
```

## AGENTS.md

Project-specific agent configuration file, auto-loaded from project root.

**Generate with:**
```bash
kimi
/init
```

**Format:**
```markdown
# Project Agents

## Agent: specialist
Role: Database optimization specialist
Context: Focus on SQL queries, indexing, migrations
Tools: +ReadFile, +Grep, Shell

## Agent: reviewer
Role: Code review specialist
Context: Security, performance, style checks
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| `Shell` | Execute shell commands |
| `ReadFile` | Read file contents |
| `WriteFile` | Write/create files |
| `StrReplaceFile` | String replacement in files |
| `Glob` | Pattern-based file search |
| `Grep` | Content search in files |
| `ReadMediaFile` | Read images/audio/video |
| `SearchWeb` | Web search |
| `FetchURL` | HTTP GET request |
| `Agent` | Spawn sub-agent |
| `AskUserQuestion` | Prompt user for input |
| `SetTodoList` | Manage task list |
| `EnterPlanMode` | Enter planning mode |
| `ExitPlanMode` | Exit planning mode |
| `TaskList` | List background tasks |
| `TaskOutput` | Get task output |
| `TaskStop` | Stop background task |

## Built-in Subagents

| Agent | Purpose | Default Tools |
|-------|---------|---------------|
| `coder` | General software engineering | All tools |
| `explore` | Read-only codebase exploration | ReadFile, Glob, Grep |
| `plan` | Implementation planning | All tools (non-executing) |

## Interactive Session: Slash Commands

| Command | Description |
|---------|-------------|
| `/help` | Show help and available commands |
| `/compact [focus]` | Compact context with optional focus |
| `/clear` | Clear all context |
| `/plan` | Enter planning mode |
| `/model [name]` | Switch model |
| `/exit` | Exit session |

## Interactive Session: Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Ctrl+C` | Cancel current operation / request confirmation to exit |
| `Ctrl+D` | Exit session |
| `Shift+Tab` | Accept auto-completion |
| `\` + `Enter` | Insert newline (don't send) |
| `Shift+Enter` | Force send incomplete message |

## Pitfalls & Gotchas

1. **Command name variation:** The binary may be installed as `kimi` or `kimi-cli` depending on installation method. Check with `which kimi` or `which kimi-cli`.

2. **No git required:** Unlike Claude Code, Kimi Code does not require a git repository. It works in any directory.

3. **Print mode safest:** For non-interactive Hermes workflows, always prefer `--print` or `--quiet` to avoid PTY-related hangs.

4. **Exit code 2 means retry:** When exit code is 2, the failure is transient (rate limit, network). Implement exponential backoff and retry.

5. **Context auto-compaction:** At ~85% of context limit, Kimi auto-compacts. This may lose some nuance from earlier conversation.

6. **File truncation risk:** When generating multiple files, earlier files may be truncated if context limit is hit. Use `--max-steps-per-turn` to mitigate.

7. **macOS Gatekeeper:** First run may pause for Gatekeeper verification. Pre-arm by running `kimi --version` once manually.

8. **API key platform mismatch:** Kimi Code (api.kimi.com) and Kimi Open Platform (api.moonshot.cn) keys are NOT interchangeable.

9. **Thinking mode compatibility:** Not all models support `--thinking`. Using with incompatible models will fail silently or error.

10. **Session resumption requires same directory:** `--continue` only works for sessions started in the current working directory.

## Rules for Hermes Agents

1. **Default to print mode:** Use `kimi --print` for all automated tasks. Reserve interactive mode for user-initiated complex workflows.

2. **Always check exit codes:** Implement retry logic for exit code 2. Abort immediately on exit code 1.

3. **Pre-approve paths when possible:** Run `kimi acp` in known project directories to reduce trust dialogs.

4. **Use worktrees for parallelism:** Never run multiple `kimi` processes in the same directory. Use git worktrees instead.

5. **Monitor context usage:** For long sessions, watch for compaction events. Consider `/compact` with focus at strategic points.

6. **Specify model explicitly:** Don't rely on default model. Use `--model` to ensure consistent behavior.

7. **Capture structured output:** Use `--output-format stream-json` for programmatic consumption. Parse the JSON stream for tool calls and results.

8. **Set resource limits:** Use `--max-steps-per-turn` and `--max-ralph-iterations` to prevent infinite loops in automated workflows.

9. **Validate API key platform:** Ensure the correct API key is set for the intended service (Code vs Open Platform).

10. **Handle first-run delays:** On macOS, account for Gatekeeper delays on first execution. Set appropriate timeouts.

11. **Use background mode for long tasks:** For tasks expected to run >5 minutes, use background mode with output redirection and process monitoring.

12. **Leverage subagents:** Use `--agent explore` for read-only tasks and `--agent plan` for design work. Reserve default `coder` for implementation.
