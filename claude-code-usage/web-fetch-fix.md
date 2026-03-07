# 解决中转 API 下 Claude Code Web Fetch 不可用的问题

使用中转 API 服务（而非 Anthropic 官方 API）运行 Claude Code 时，你可能会发现 Web Fetch 工具无法正常工作。本文分析两个根因并给出解决方案。

## 问题现象

在 Claude Code 中使用 Web Fetch 工具抓取网页时，报错或无响应。

## 原因分析

Web Fetch 内部有两个步骤，各自可能出问题：

### 原因一：Preflight 安全检查访问 claude.ai 失败

Claude Code 在发起 Web Fetch 前，会先访问 `claude.ai` 确认目标网站是否安全。然而在美国以外的地区，直接访问 `claude.ai` 可能会被阻断或返回错误，导致 preflight 检查失败，Web Fetch 无法继续。

### 原因二：内置 Haiku 小模型不可用

Web Fetch 抓取到网页原始内容后，会调用 Claude 自家的小模型（Haiku）来做内容解析和提取。如果你的中转 API 服务商：

- 没有提供 Haiku 模型
- 或者模型路径/名称与 Claude Code 默认的不一致

就会导致解析步骤失败，Web Fetch 返回空结果或报错。

## 解决方案

编辑 `~/.claude/settings.json`，添加以下两项配置。

### 第 1 步：跳过 Preflight 检查

在 `settings.json` 顶层添加：

```json
{
  "skipWebFetchPreflight": true
}
```

这会让 Claude Code 跳过对 `claude.ai` 的安全预检，直接发起 Web Fetch 请求。

### 第 2 步：指定可用的 Haiku 模型

在 `settings.json` 的 `env` 字段中添加环境变量，将 Haiku 模型替换为你的中转服务商提供的可用模型路径：

```json
{
  "env": {
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "your-path/claude-haiku-4-5-20251001"
  }
}
```

> 将 `your-path/claude-haiku-4-5-20251001` 替换为你的中转 API 实际支持的模型名称。不确定的话，可以查阅中转服务商的模型列表文档。

### 完整配置示例

假设你的中转 API 地址为 `https://api.example.com/anthropic`，Haiku 模型路径为 `pa/claude-haiku-4-5-20251001`，那么 `~/.claude/settings.json` 的相关部分如下：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.example.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "your-api-key",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "pa/claude-haiku-4-5-20251001"
  },
  "skipWebFetchPreflight": true
}
```

### 第 3 步：重启 Claude Code

配置保存后，关闭并重新打开 Claude Code 会话，即可生效。

## 验证

在 Claude Code 中尝试使用 Web Fetch 访问一个网页，例如：

> 帮我抓取 https://example.com 的内容

如果返回了正常的网页内容摘要，说明配置成功。

## 总结

| 问题 | 原因 | 解决配置 |
|------|------|----------|
| Preflight 检查失败 | 美国以外地区无法访问 `claude.ai` | `"skipWebFetchPreflight": true` |
| 网页内容解析失败 | 中转服务的 Haiku 模型路径不匹配 | `"ANTHROPIC_DEFAULT_HAIKU_MODEL": "..."` |
