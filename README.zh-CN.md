# Hermes Skill: Kimi Code

[English](README.md) | 简体中文

用于将编码任务委托给 [Kimi Code CLI](https://www.kimi.com/code/) 的 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 技能 —— Moonshot AI 的自主编码代理。

## Kimi Code 是什么？

Kimi Code 是 Moonshot AI 推出的终端编码代理，能够：
- 读取和编辑代码文件
- 执行 shell 命令
- 搜索和抓取网页
- 自主规划和调整行动
- 生成子代理执行专项任务

## 本技能提供什么

本技能教会 Hermes Agent 如何编排 Kimi Code CLI，实现：
- 一次性编码任务（打印模式）
- 交互式多轮会话（tmux）
- 长时间后台任务
- PR 代码审查
- 使用 git worktrees 并行工作
- 会话管理和恢复

## 如何在 Hermes 中触发此技能？

触发短语包括：
- "Use Kimi Code to refactor the auth module"（使用 Kimi Code 重构认证模块）
- "Ask Kimi to fix this bug"（让 Kimi 修复这个 bug）
- "Have Kimi review the PR"（让 Kimi 审查 PR）
- "Let Kimi Code handle this"（让 Kimi Code 处理这个）
- "Use kimi-cli for this task"（对此任务使用 kimi-cli）
- "Defer to Kimi Code CLI"（委托给 Kimi Code CLI）

## 安装

### 安装技能

```bash
# 克隆到 Hermes skills 目录
git clone https://github.com/xiongye/hermes-skill-kimi-code.git ~/.hermes/skills/autonomous-ai-agents/kimi-code

# 或使用 Hermes skill 管理器
hermes skills install xiongye/hermes-skill-kimi-code
```

### 安装 Kimi Code CLI

```bash
# macOS / Linux
curl -LsSf https://code.kimi.com/install.sh | bash

# 或通过 uv 安装
uv tool install --python 3.13 kimi-cli
```

### 认证登录

```bash
kimi
/login
```

按提示完成 OAuth 或 API Key 设置。

## 快速开始

### 一次性任务（打印模式）

```bash
kimi --print -p "给 src/ 下的所有 API 调用添加错误处理"
```

### 交互式会话

```bash
cd ~/my-project
kimi
```

### 后台任务

```bash
kimi --print -p "重构认证模块，使用 JWT 令牌" &
```

## 文档

完整的编排指南请见 [SKILL.md](skills/kimi-code/SKILL.md)。

## 相关技能

- [claude-code](https://github.com/NousResearch/hermes-agent/tree/main/skills/autonomous-ai-agents/claude-code) — Anthropic 的 Claude Code CLI
- [codex](https://github.com/NousResearch/hermes-agent/tree/main/skills/autonomous-ai-agents/codex) — OpenAI 的 Codex CLI
- [opencode](https://github.com/NousResearch/hermes-agent/tree/main/skills/autonomous-ai-agents/opencode) — OpenCode CLI

## 开源协议

[Apache 2.0](LICENSE) — 详见 [LICENSE](LICENSE) 文件。
