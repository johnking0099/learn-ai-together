# Claude Code 使用技巧

使用中转 API 运行 Claude Code 时的实用配置与问题解决指南。

## 目录

| 文章 | 说明 |
|------|------|
| [解决 Web Fetch 不可用](web-fetch-fix.md) | 中转 API 下 Web Fetch 失败的两个原因及修复方法 |
| [解决 Web Search 不可用](web-search-fix.md) | 用 Tavily MCP 替代内置 Web Search 服务 |
| [自定义 Status Line](statusline.md) | 配置终端底部状态栏，实时显示模型、token、费用等信息 |
| [解决 Agent Team 模型不可用](agent-team-fix.md) | 中转 API 下 team member 模型解析失败的原因及 modelOverrides 修复方案 |
| [免费无限制的 Web Search](free-search.md) | 本地部署 SearXNG + MCP，替代有调用限制的付费搜索服务 |

## 适用场景

这些教程主要面向通过**中转 API 服务商**（而非 Anthropic 官方 API）使用 Claude Code 的用户。如果你直接使用官方 API，大部分功能开箱即用，不需要额外配置。

## 核心配置文件

所有配置都在同一个文件中完成：

```
~/.claude/settings.json
```

一个包含本系列所有配置项的完整示例：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.example.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "your-api-key",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "pa/claude-haiku-4-5-20251001",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "pa/claude-opus-4-6",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "pa/claude-sonnet-4-6"
  },
  "modelOverrides": {
    "claude-opus-4-6": "pa/claude-opus-4-6",
    "claude-sonnet-4-6": "pa/claude-sonnet-4-6",
    "claude-haiku-4-5-20251001": "pa/claude-haiku-4-5-20251001"
  },
  "permissions": {
    "deny": ["WebSearch"]
  },
  "skipWebFetchPreflight": true,
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```
