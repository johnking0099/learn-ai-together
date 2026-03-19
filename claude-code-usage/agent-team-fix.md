# 解决中转 API 下 Claude Code Agent Team 模型不可用的问题

使用中转 API 服务运行 Claude Code 时，启动 Agent Team 的成员（teammate）可能报错：

```
There's an issue with the selected model (claude-opus-4-6).
It may not exist or you may not have access to it.
```

本文分析根因并给出解决方案。

## 问题现象

通过 `TeamCreate` 创建团队后，用 `Agent` 工具 spawn 的 team member 启动失败，提示模型 `claude-opus-4-6` 不存在或无权访问。

而主会话使用相同的模型却一切正常。

## 原因分析

### 主会话为什么能工作

主会话的模型解析链：

1. `settings.json` 中设置 `"model": "opus[1m]"`
2. Claude Code 查找 `ANTHROPIC_DEFAULT_OPUS_MODEL` 环境变量
3. 解析为 `pa/claude-opus-4-6`（中转 API 需要的带 `pa/` 前缀的模型 ID）
4. 拼接上下文窗口后缀，最终发送 `pa/claude-opus-4-6[1m]` 给 API

### Team member 为什么失败

Team member 是**独立的 Claude Code 会话**（不是普通的 subagent），它的模型解析走了不同的路径：

1. 继承父进程的模型设置 `opus[1m]`
2. 解析时可能**绕过** `ANTHROPIC_DEFAULT_OPUS_MODEL` 映射
3. 退回到标准模型 ID `claude-opus-4-6`
4. 中转 API 不认识这个 ID，报错

### 关键区别：Team member ≠ 普通 Subagent

| 类型 | 本质 | 模型控制方式 |
|------|------|------------|
| 普通 Subagent（Agent 工具） | 主进程内的子任务 | `CLAUDE_CODE_SUBAGENT_MODEL` 环境变量 |
| Team member（Agent + team_name） | 独立的 Claude Code 会话 | 继承主会话模型，走独立解析 |

所以即使设置了 `CLAUDE_CODE_SUBAGENT_MODEL`，对 team member 也不生效。

## 解决方案：使用 modelOverrides

`modelOverrides` 是 Claude Code 专门为代理/网关场景设计的配置项，在 **API 请求发送前**重映射模型 ID。无论模型解析走哪条路径，最终发出的请求都会被拦截和修正。

编辑 `~/.claude/settings.json`，添加 `modelOverrides`：

```json
{
  "model": "opus[1m]",
  "modelOverrides": {
    "claude-opus-4-6": "pa/claude-opus-4-6",
    "claude-sonnet-4-6": "pa/claude-sonnet-4-6",
    "claude-haiku-4-5-20251001": "pa/claude-haiku-4-5-20251001"
  },
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.example.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "your-api-key",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "pa/claude-opus-4-6",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "pa/claude-sonnet-4-6",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "pa/claude-haiku-4-5-20251001"
  }
}
```

将 `pa/` 替换为你的中转 API 服务商要求的模型前缀。

### 为什么选 modelOverrides 而不是 ANTHROPIC_DEFAULT_*_MODEL

两者作用在不同阶段：

| 机制 | 作用时机 | 覆盖范围 |
|------|---------|---------|
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 模型名解析阶段 | 主会话 + 继承 env 的子进程 |
| `modelOverrides` | API 请求发送前 | **所有会话**（包括 team member） |

根据文档，通过 `ANTHROPIC_DEFAULT_*_MODEL` 解析的模型 ID 会**绕过** `modelOverrides`，直接传给 API。所以两者不会冲突——保留 `ANTHROPIC_DEFAULT_*_MODEL` 作为主会话的快速解析路径，`modelOverrides` 作为 team member 的兜底保障。

### 修改后需要重启

`settings.json` 的变更需要**重启 Claude Code 会话**才能生效。

## 补充：Team member 的终端后端

Agent Team 的 member 需要一个终端来运行，Claude Code 会自动检测可用的后端：

| 后端 | 条件 | 表现 |
|------|------|------|
| tmux | Claude Code 在 tmux session 内启动 | tmux 分屏 |
| iTerm2 | 在 iTerm2 中运行且未在 tmux 内 | iTerm2 分屏 |
| in-process | 以上都不可用 | 后台进程，无可视化窗口 |

通过 `teammateMode` 设置可以控制行为：

```json
{
  "teammateMode": "auto"
}
```

- `"auto"`（默认）：在 tmux 内用 tmux，否则用 iTerm2
- `"tmux"`：强制分屏模式，自动检测 tmux 或 iTerm2
- `"in-process"`：不分屏，全部后台运行

如果你不在 tmux 内运行且使用 iTerm2，默认行为已经是 iTerm2 分屏，无需额外配置。

## 额外问题：Spawn Teammate 时不设 model 参数导致卡死

即使配置了 `modelOverrides`，Agent Team 仍可能出现 teammate 卡死的情况。

### 现象

使用 `Agent` 工具 spawn teammate 时，如果没有显式设置 `model` 参数，teammate 会启动但**完全无法工作**：

- 任务始终停留在 `pending` 状态，不会被认领
- 向 teammate 发送消息没有任何响应
- Shutdown 请求也无法被处理
- 最终只能手动删除 team 文件清理

### 原因

查看 teammate 的启动命令：

```bash
claude --agent-id test-worker@my-team --model claude-opus-4-6 ...
```

即使你没设置 `model` 参数，Claude Code 仍然会自动填入 `--model claude-opus-4-6`（原始模型 ID）。**通过 `--model` 命令行参数传入的原始模型 ID 不经过 `modelOverrides` 处理**，所以 teammate 连不上中转 API，直接卡死。

而显式设置 `model: "opus"` 时，传入的是别名，会经过 `modelOverrides` 映射为正确的中转模型 ID，teammate 正常工作。

### 解决方法

让 Claude Code 添加一条**全局记忆**，确保每次创建 Agent Team 时都使用正确的 model 参数。

对 Claude Code 说：

> 请记住以下内容，保存到全局记忆中：使用 Agent Team 时，因为用户可能使用 API 中转服务，模型名字需要通过 modelOverrides 做映射。所以 spawn teammate 时，必须始终显式传递 `"opus"`、`"sonnet"` 或 `"haiku"` 作为 model 参数，而不能省略或使用原始模型 ID，否则 modelOverrides 不会生效，teammate 将无法连接到正确的模型。

Claude Code 会将此写入 `~/.claude/memory/MEMORY.md`，之后所有项目的会话都会自动加载这条规则。

## 验证

修改配置并重启后，创建一个简单的 team 测试：

1. 创建团队和任务
2. Spawn 一个 team member，确认 Claude Code 自动设置了 `model: "opus"`
3. 观察 member 是否成功启动并完成任务

如果 member 成功运行且不再报模型错误，说明配置生效。
