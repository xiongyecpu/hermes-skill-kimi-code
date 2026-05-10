# Kimi Code — Hermes 技能

通过 Hermes 终端将编码任务委派给 [Kimi Code CLI](https://www.kimi.com/code/)（月之暗面的自主编码代理）。Kimi Code CLI 可以读/写代码、执行 Shell 命令、搜索/获取网页内容，并自主规划和调整操作。

## 在 Hermes 中怎么触发

你可以使用以下自然语言短语来调用此技能：

- "让 Kimi 重构一下认证模块"
- "用 Kimi Code 修复 user_service.py 里的 bug"
- "让 Kimi 审查一下这个 PR 的变更"
- "让 Kimi 处理数据库迁移"
- "让 Kimi 为这个代码库添加单元测试"
- "启动一个 Kimi 会话来清理前端代码"
- "后台任务：Kimi 实现支付功能"
- "交互式 Kimi：调试 API 延迟问题"

## 安装

### 1. 安装 Kimi Code CLI

选择以下任一方式：

**macOS/Linux (curl):**
```bash
curl -LsSf https://code.kimi.com/install.sh | bash
```

**Windows PowerShell:**
```powershell
Invoke-RestMethod https://code.kimi.com/install.ps1 | Invoke-Expression
```

**使用 uv（推荐给开发者）:**
```bash
uv tool install --python 3.13 kimi-cli
```

### 2. 认证

运行 `kimi`，然后输入 `/login` 来配置你的账户（OAuth 或 API Key）。

```bash
kimi
> /login
```

### 3. 验证安装

```bash
kimi --version
```

### 4. 安装此技能

将 `kimi-code` 技能文件夹复制到你的 Hermes 技能目录：

```bash
cp -r skills/kimi-code ~/.hermes/skills/
```

## 快速开始

### 一次性任务（打印模式）

运行单个任务并获取结果：

```
terminal(command="kimi --print -p '为 src/ 中所有 API 调用添加错误处理'", workdir="/path/to/project", timeout=120)
```

### 交互式会话（tmux）

启动多轮会话进行迭代工作：

```
terminal(command="tmux new-session -d -s kimi-work -x 140 -y 40")
terminal(command="tmux send-keys -t kimi-work 'cd /path/to/project && kimi' Enter")
terminal(command="sleep 5 && tmux send-keys -t kimi-work '将认证模块重构为使用 JWT token' Enter")
```

### 后台任务

在后台运行耗时任务：

```
terminal(command="kimi --print -p '实现 plan.md 中描述的功能'", workdir="~/project", background=true, timeout=600)
```

## 主要特性

- **打印模式** (`--print`) — 非交互式，自动批准所有操作
- **交互式模式** — 通过 tmux 实现完整的对话式 REPL
- **PR 审查** — 自动化代码审查与差异分析
- **并行工作** — 使用 git worktree 同时运行多个任务
- **会话管理** — 使用 `--continue` 恢复之前的会话
- **MCP 支持** — 通过外部工具和服务器进行扩展
- **多提供商** — 支持 Kimi、OpenAI、Anthropic、Gemini、VertexAI

## 文档

- **[SKILL.md](./SKILL.md)** — Hermes 代理完整编排指南
- **[Kimi Code 官方文档](https://www.kimi.com/code/docs)** — 官方文档
- **[AGENTS.md](https://www.kimi.com/code/docs/project-context)** — 项目上下文格式

## 相关技能

- **[claude-code](../claude-code/)** — Claude Code CLI 集成
- **[codex](../codex/)** — Codex AI 集成
- **[opencode](../opencode/)** — OpenCode 集成

## 许可证

Apache License 2.0

## English Documentation

[English version](./README.md)
