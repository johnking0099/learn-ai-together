# 解决 Claude Code 内置 Web Search 不可用的问题

## 问题现象

在 Claude Code 中使用 web search 时，输出类似如下内容：

```
⏺ Web Search("today's top technology news March 2026")
  ⎿  Did 0 searches in 8s
```

搜索结果为 0，说明内置 web search 服务不可用。

## 原因分析

Claude Code 内置的 web search 是一项**付费服务**，使用它需要**同时满足**两个条件：

1. **客户端网络在美国** — 发起请求的机器需要位于美国网络环境
2. **订阅了 Claude Code 官方的 web search 服务** — 这是 Anthropic 提供的收费功能

因此，以下两种情况 web search 都会不可用：

- 你在**美国以外的网络**使用 Claude Code
- 你使用的是**中转 API 服务**而非 Anthropic 官方订阅

## 解决方案：用 Tavily MCP 替代内置 Web Search

思路很简单：既然内置的 web search 用不了，就通过 MCP（Model Context Protocol）接入第三方搜索服务来替代。这里以 [Tavily](https://www.tavily.com/) 为例。

### 第 1 步：注册 Tavily 获取 API Key

1. 访问 [https://www.tavily.com/](https://www.tavily.com/)，注册一个账户
2. 登录后，在产品的 **Overview** 页面找到 **API Keys**
3. 复制你的 API Key

### 第 2 步：为 Claude Code 添加 Tavily MCP

执行以下命令，将 `YOUR_API_KEY` 替换为你在上一步获取的 API Key：

```bash
claude mcp add -s user -t http tavily-search "https://mcp.tavily.com/mcp/?tavilyApiKey=YOUR_API_KEY"
```

添加完成后，验证 MCP server 是否连接正常：

```bash
claude mcp list
```

看到类似如下输出，表示连接成功：

```
Checking MCP server health...

tavily-search: https://mcp.tavily.com/mcp/?tavilyApiKey=tvly-dev-xxx (HTTP) - ✓ Connected
```

确认 `tavily-search` 状态为 `✓ Connected`，即可在 Claude Code 中使用 Tavily 的搜索服务了。

> **配置文件说明**：`claude mcp add -s user` 命令会将 MCP 配置写入用户级配置文件 `~/.claude.json`（注意不是 `~/.claude/settings.json`）。写入后的内容如下：
>
> ```json
> {
>   "mcpServers": {
>     "tavily-search": {
>       "type": "http",
>       "url": "https://mcp.tavily.com/mcp/?tavilyApiKey=tvly-dev-xxx"
>     }
>   }
> }
> ```
>
> 你也可以直接编辑 `~/.claude.json` 手动添加 MCP server，效果相同。

### 第 3 步：禁用内置 Web Search（推荐）

完成上面两步后，Tavily search 已经可以正常工作。但 Claude Code 在使用过程中**有时仍会优先尝试内置的 web search**，失败后才会回退到 MCP 的搜索服务，这会浪费时间。

解决方法是在配置中显式禁用内置 web search。编辑 `~/.claude/settings.json`，添加：

```json
{
  "permissions": {
    "deny": [
      "WebSearch"
    ]
  }
}
```

> **注意**：如果你的 `settings.json` 中已有 `permissions` 字段，将 `"WebSearch"` 追加到现有的 `deny` 数组中即可。

保存后重启 Claude Code，它就不会再尝试使用内置 web search，而是直接使用 Tavily MCP 进行搜索。

## 完整配置示例

本方案涉及两个配置文件：

**`~/.claude/settings.json`** — 禁用内置 Web Search：

```json
{
  "permissions": {
    "deny": [
      "WebSearch"
    ]
  }
}
```

**`~/.claude.json`** — Tavily MCP server 配置（由 `claude mcp add -s user` 自动写入）：

```json
{
  "mcpServers": {
    "tavily-search": {
      "type": "http",
      "url": "https://mcp.tavily.com/mcp/?tavilyApiKey=tvly-dev-xxx"
    }
  }
}
```

## 总结

| 步骤 | 操作 | 目的 |
|------|------|------|
| 1 | 注册 Tavily，获取 API Key | 准备第三方搜索服务 |
| 2 | `claude mcp add` 添加 Tavily MCP | 让 Claude Code 获得搜索能力 |
| 3 | `permissions.deny` 中添加 `WebSearch` | 避免 Claude Code 仍尝试失败的内置搜索 |
