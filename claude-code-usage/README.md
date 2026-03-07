# Claude Code 使用技巧

使用中转 API 运行 Claude Code 时的实用配置与问题解决指南。

## 目录

| 文章 | 说明 |
|------|------|
| [解决 Web Fetch 不可用](web-fetch-fix.md) | 中转 API 下 Web Fetch 失败的两个原因及修复方法 |
| [解决 Web Search 不可用](web-search-fix.md) | 用 Tavily MCP 替代内置 Web Search 服务 |
| [自定义 Status Line](statusline.md) | 配置终端底部状态栏，实时显示模型、token、费用等信息 |

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
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "pa/claude-haiku-4-5-20251001"
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
