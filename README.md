# pi-subagent

将任务委派给专门的子代理（subagent），每个子代理运行在独立的 `pi` 进程中，拥有独立的上下文窗口。

## 安装

### 通过 pi 包管理器安装（推荐）

```bash
# 全局安装
pi install git:github.com/ekil1100/pi-subagent

# 或项目本地安装
pi install git:github.com/ekil1100/pi-subagent -l

# 指定版本标签
pi install git:github.com/ekil1100/pi-subagent@v1.0.0
```

### 安装 Agent 定义

**Agent 定义需要手动安装**，因为 pi 目前不通过包管理器自动分发 Agent：

```bash
# 找到包安装路径
PKG_PATH=~/.pi/agent/git/github.com/ekil1100/pi-subagent

# 全局安装 agents
mkdir -p ~/.pi/agent/agents
cp "$PKG_PATH"/agents/*.md ~/.pi/agent/agents/

# 或项目本地安装
mkdir -p .pi/agents
cp .pi/git/github.com/ekil1100/pi-subagent/agents/*.md .pi/agents/
```

> **注意**：pi 目前不支持通过包自动安装 agents。你需要将 `agents/*.md` 文件手动复制到 `~/.pi/agent/agents/`（全局）或 `.pi/agents/`（项目本地）。

## 功能

- **隔离上下文**：每个子代理运行在单独的 `pi` 进程中，不污染主会话
- **流式输出**：实时查看子代理的工具调用和进度
- **并行执行**：可同时运行多个子代理（最多 8 个任务，4 个并发）
- **链式工作流**：支持顺序执行，上一步的输出通过 `{previous}` 传递给下一步
- **用量追踪**：显示每个代理的轮数、token、费用和上下文使用量
- **中断支持**：Ctrl+C 会传播到子代理进程并终止它们

## 使用

### 单个子代理

```
Use scout to find all authentication code
```

### 并行执行

```
Run 2 scouts in parallel: one to find models, one to find providers
```

### 链式工作流

```
Use a chain: first have scout find the read tool, then have planner suggest improvements
```

### 工作流预设（Prompt Templates）

安装后可直接使用：

```
/implement add Redis caching to the session store
/scout-and-plan refactor auth to support OAuth
/implement-and-review add input validation to API endpoints
```

## 预设 Agent

| Agent | 用途 | 模型 | 工具 |
|-------|------|------|------|
| `scout` | 快速代码侦察 | Haiku | read, grep, find, ls, bash |
| `planner` | 制定实现计划 | Sonnet | read, grep, find, ls |
| `reviewer` | 代码审查 | Sonnet | read, grep, find, ls, bash |
| `worker` | 通用工作代理 | Sonnet | （全部默认工具） |

## 工具模式

| 模式 | 参数 | 说明 |
|------|------|------|
| 单任务 | `{ agent, task }` | 一个 Agent，一个任务 |
| 并行 | `{ tasks: [...] }` | 多个 Agent 并发执行（最多 8 个，4 个并发） |
| 链式 | `{ chain: [...] }` | 顺序执行，支持 `{previous}` 占位符 |

## 参数说明

```typescript
{
  agent?: string;           // 单任务模式：Agent 名称
  task?: string;            // 单任务模式：任务描述
  tasks?: Array<{           // 并行模式
    agent: string;
    task: string;
    cwd?: string;           // 可选：工作目录
  }>;
  chain?: Array<{           // 链式模式
    agent: string;
    task: string;           // 可用 {previous} 引用上一步输出
    cwd?: string;
  }>;
  agentScope?: "user" | "project" | "both";  // 默认 "user"
  confirmProjectAgents?: boolean;             // 运行项目级 Agent 前确认，默认 true
  cwd?: string;             // 单任务模式的工作目录
}
```

## 自定义 Agent

在 `~/.pi/agent/agents/`（全局）或 `.pi/agents/`（项目本地）下创建新的 `.md` 文件：

```markdown
---
name: tester
description: Write unit tests for the given code
tools: read, write, bash
model: claude-sonnet-4-5
---

You are a test engineer. Write comprehensive unit tests for the code provided.
Use the project's testing framework. Run tests to verify they pass.
```

然后即可使用：

```
Use tester to write tests for src/utils.ts
```

## Agent 定义格式

Agent 是带 YAML frontmatter 的 Markdown 文件：

```markdown
---
name: my-agent
description: What this agent does
tools: read, grep, find, ls
model: claude-haiku-4-5
---

System prompt for the agent goes here.
```

**字段说明：**
- `name`：Agent 的唯一标识（必填）
- `description`：Agent 的描述（必填）
- `tools`：（可选）该 Agent 可使用的工具，逗号分隔
- `model`：（可选）指定模型，如 `claude-sonnet-4-5`

## 安全模型

- 默认只加载用户级 Agent（`~/.pi/agent/agents/`）
- 项目级 Agent（`.pi/agents/`）需要显式设置 `agentScope: "both"`
- 交互模式下，运行项目级 Agent 前会弹出确认对话框
- 子代理以独立的 `pi` 子进程运行，拥有完整的系统权限

## 快捷键

- **Ctrl+O**：在 TUI 中展开/折叠子代理输出
- **Escape / Ctrl+C**：中断正在运行的子代理

## 更新

```bash
pi update
```

## 卸载

```bash
pi remove git:github.com/ekil1100/pi-subagent
```

## 参考

基于 [pi 官方 subagent 示例](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent/examples/extensions/subagent) 构建。
