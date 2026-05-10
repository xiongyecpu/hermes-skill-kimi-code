# Hermes Skill: Kimi Code

A [Hermes Agent](https://github.com/NousResearch/hermes-agent) skill for delegating coding tasks to [Kimi Code CLI](https://www.kimi.com/code/) — Moonshot AI's autonomous coding agent.

## What is Kimi Code?

Kimi Code is Moonshot AI's coding agent that runs in your terminal. It can:
- Read and edit code files
- Execute shell commands
- Search and fetch web pages
- Autonomously plan and adjust actions
- Spawn subagents for specialized tasks

## What this skill provides

This skill teaches Hermes Agent how to orchestrate Kimi Code CLI for:
- One-shot coding tasks (print mode)
- Interactive multi-turn sessions (tmux)
- Long-running background tasks
- PR reviews
- Parallel work with git worktrees
- Session management and resumption

## How do I trigger this skill in Hermes?

Trigger phrases include:
- "Use Kimi Code to refactor the auth module"
- "Ask Kimi to fix this bug"
- "Have Kimi review the PR"
- "Let Kimi Code handle this"
- "Use kimi-cli for this task"
- "Defer to Kimi Code CLI"

## Installation

### Install the skill

```bash
# Clone to your Hermes skills directory
git clone https://github.com/xiongye/hermes-skill-kimi-code.git ~/.hermes/skills/autonomous-ai-agents/kimi-code

# Or use Hermes skill manager
hermes skills install xiongye/hermes-skill-kimi-code
```

### Install Kimi Code CLI

```bash
# macOS / Linux
curl -LsSf https://code.kimi.com/install.sh | bash

# Or via uv
uv tool install --python 3.13 kimi-cli
```

### Authenticate

```bash
kimi
/login
```

Follow the prompts to complete OAuth or API key setup.

## Quick Start

### One-shot task (print mode)

```bash
kimi --print -p "Add error handling to all API calls in src/"
```

### Interactive session

```bash
cd ~/my-project
kimi
```

### Background task

```bash
kimi --print -p "Refactor the auth module to use JWT tokens" &
```

## Documentation

See [SKILL.md](skills/kimi-code/SKILL.md) for the complete orchestration guide.

## Related Skills

- [claude-code](https://github.com/NousResearch/hermes-agent/tree/main/skills/autonomous-ai-agents/claude-code) — Anthropic's Claude Code CLI
- [codex](https://github.com/NousResearch/hermes-agent/tree/main/skills/autonomous-ai-agents/codex) — OpenAI's Codex CLI
- [opencode](https://github.com/NousResearch/hermes-agent/tree/main/skills/autonomous-ai-agents/opencode) — OpenCode CLI

## License

[Apache 2.0](LICENSE) — see [LICENSE](LICENSE) file for details.

---

[简体中文](README.zh-CN.md)
