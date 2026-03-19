# 免费无限制的 Web Search 方案：本地部署 SearXNG

## 背景

在 [解决 Web Search 不可用](web-search-fix.md) 中，我们用 Tavily MCP 替代了 Claude Code 内置的 Web Search。但 Tavily 免费额度只有 **1,000 次/月**，在频繁使用 Agent 的场景下很快就会耗尽（甚至会触发 429 限流）。

其他主流搜索 MCP 也有类似限制：

| 服务 | 免费额度 |
|------|----------|
| Tavily | 1,000 次/月 |
| Brave Search | 1,000 次/月 |
| Exa | 已转为付费 |

**本文介绍一种完全免费、无调用限制的方案**：本地部署 SearXNG，配合 MCP server 使用。

## 什么是 SearXNG

[SearXNG](https://github.com/searxng/searxng) 是一个开源的**元搜索引擎**（meta search engine）。它本身不维护搜索索引，而是将搜索请求转发给多个搜索引擎（Google、Bing、DuckDuckGo、Brave 等），汇总去重后返回结果。

核心特点：

- **自托管**：部署在本地，完全由你掌控
- **无调用限制**：没有第三方的 API 配额限制
- **隐私友好**：不追踪用户，不记录搜索历史
- **多引擎聚合**：可配置 70+ 个搜索引擎源

## 部署 SearXNG

### 前提条件

- 已安装 Docker

### 第 1 步：创建配置文件

```bash
mkdir -p ~/searxng
```

创建 `~/searxng/settings.yml`，写入以下内容：

```yaml
use_default_settings: true

search:
  formats:
    - html
    - json

server:
  secret_key: "a8b2c9d4e5f1"
  limiter: false

engines:
  - name: bing
    disabled: false
  - name: yahoo
    disabled: false
  - name: baidu
    disabled: false
  - name: sogou
    disabled: false
  - name: yandex
    disabled: false
```

关键配置说明：

| 配置项 | 说明 |
|--------|------|
| `use_default_settings: true` | 继承默认配置，只需写需要覆盖的项 |
| `formats: [html, json]` | **必须添加 json**，否则 API 调用会返回 403 |
| `secret_key` | SearXNG 用于加密 Cookie 和防伪造请求的密钥，必须设置，填一个随意的字符串即可 |
| `limiter: false` | 关闭限流器，否则非浏览器请求会被 429 拦截 |
| `engines` | 额外启用的搜索引擎。默认只开了 brave、duckduckgo、google、startpage 四个，这里补充 bing、yahoo、baidu、sogou、yandex 以增加结果覆盖面（尤其是中文内容）。SearXNG 并行请求所有引擎，某个失败不影响其他结果 |

### 第 2 步：启动容器

```bash
docker run -d \
  --name searxng \
  --restart always \
  -p 8888:8080 \
  -v ~/searxng:/etc/searxng:rw \
  searxng/searxng
```

参数说明：

| 参数 | 说明 |
|------|------|
| `--name searxng` | 容器命名，方便管理 |
| `--restart always` | 自动重启，开机自启 |
| `-p 8888:8080` | 宿主机 8888 端口映射到容器 8080 |
| `-v ~/searxng:/etc/searxng:rw` | 挂载配置目录，配置持久化 |

### 第 3 步：验证部署

等待几秒后，测试 JSON API 是否可用：

```bash
curl -s "http://localhost:8888/search?q=test&format=json" | python3 -m json.tool | head -20
```

看到 JSON 格式的搜索结果即表示部署成功。

## 配合 MCP Server 使用

SearXNG 部署好后，需要一个 MCP server 作为 Claude Code 和 SearXNG 之间的桥梁。推荐使用 [mcp-searxng](https://github.com/ihor-sokoliuk/mcp-searxng)，社区最活跃、文档完善、支持 STDIO 和 HTTP 双传输模式。

### 安装 MCP Server

```bash
npm install -g mcp-searxng
```

### 添加到 Claude Code

```bash
claude mcp add searxng -s user -e "SEARXNG_URL=http://localhost:8888" -- mcp-searxng
```

> **注意**：`claude mcp add` 的参数顺序比较特殊——name（这里是 `searxng`）必须紧跟在 `add` 后面，否则会报 `missing required argument` 错误。

验证连接：

```bash
claude mcp list
```

确认 `searxng` 状态为 `✓ Connected` 即可。

### MCP 提供的工具

mcp-searxng 注册了两个工具，Claude 会自动按需调用：

**`searxng_web_search`** — 搜索

| 参数 | 类型 | 说明 |
|------|------|------|
| `query` | string | 搜索关键词 |
| `pageno` | number, 可选 | 页码，默认 1 |
| `time_range` | string, 可选 | 时间过滤：`day`、`month`、`year` |
| `language` | string, 可选 | 语言代码，如 `zh`、`en`，默认 `all` |
| `safesearch` | number, 可选 | 安全搜索级别：0 关闭 / 1 适度 / 2 严格 |

**`web_url_read`** — 读取网页内容并转为 Markdown

| 参数 | 类型 | 说明 |
|------|------|------|
| `url` | string | 要读取的网页 URL |
| `startChar` | number, 可选 | 从第几个字符开始提取，默认 0 |
| `maxLength` | number, 可选 | 最多返回多少字符 |
| `section` | string, 可选 | 只提取某个标题下的内容 |
| `paragraphRange` | string, 可选 | 提取指定段落范围，如 `1-5`、`3`、`10-` |
| `readHeadings` | boolean, 可选 | 只返回标题列表，不返回正文 |

`web_url_read` 弥补了搜索结果只有摘要的不足——Claude 搜到感兴趣的链接后，会自动调用它抓取完整页面内容。

### 优化搜索质量：添加全局记忆

SearXNG 搜索返回的是摘要片段（一两句话），不如 Tavily 那样直接返回页面正文。但 mcp-searxng 提供了 `web_url_read` 工具可以抓取完整内容——关键是让 Claude **每次搜索后自动去读取详情**。

在 Claude Code 中输入以下指令，添加一条全局记忆：

```
添加全局记忆，当使用 searxng_web_search 搜索后，自动从结果中选择最相关的 3 个 URL，用 web_url_read 读取8k内容，然后综合所有信息回答。
```

添加后，Claude 的搜索行为会变成：

1. 调用 `searxng_web_search` 获取搜索结果列表
2. 从结果中选出最相关的 3 个 URL
3. 对每个 URL 调用 `web_url_read`（maxLength: 8000）读取正文
4. 综合搜索摘要 + 全文内容给出完整回答

效果示例——输入"帮我搜索今天的AI新闻"后，Claude 会自动执行：

```
# 第 1 步：搜索
⏺ searxng - searxng_web_search (query: "AI人工智能新闻 2026年3月19日", time_range: "day", language: "zh")
  ⎿  Title: 芝麻AI速递:昨夜今晨财经热点要闻...
     Title: 2026年3月19日人工智能早间新闻...
     Title: AI内参 晨报...
     ... 共返回多条搜索结果（仅含标题和摘要）

# 第 2 步：自动读取最相关的 3 个页面
⏺ searxng - web_url_read (url: "https://blog.csdn.net/.../159238969", maxLength: 8000)
⏺ searxng - web_url_read (url: "https://www.neican.ai/morningnews/...", maxLength: 8000)
⏺ searxng - web_url_read (url: "https://finance.eastmoney.com/...", maxLength: 8000)

# 第 3 步：综合所有信息，输出完整回答
⏺ 以下是2026年3月19日AI领域的重要新闻综合：
  - 阿里云、百度云同日宣布AI算力涨价...
  - 黄仁勋在GTC大会称OpenClaw"绝对是下一个ChatGPT"...
  - 小米深夜上线三大自研MiMo-V2系列模型...
  ...
```

没有这条记忆时，Claude 只会返回搜索摘要（一两句话）；加了之后，会自动抓取原文再综合回答，信息量接近 Tavily，且完全免费无限制。

### 替换 Tavily（可选）

如果之前配置过 Tavily MCP，可以移除它：

```bash
claude mcp remove -s user tavily-search
```

## 常用管理命令

```bash
# 查看容器状态
docker ps --filter name=searxng

# 查看日志
docker logs searxng

# 重启（修改 settings.yml 后需要重启生效）
docker restart searxng

# 停止
docker stop searxng

# 删除容器（配置文件在宿主机上，不会丢失）
docker rm searxng
```

## 为什么不用公共 SearXNG 实例

[searx.space](https://searx.space) 列出了大量公共 SearXNG 实例，但实测几乎都**不能**用于 API 调用：

- 绝大多数实例开启了 `limiter`，脚本请求直接返回 429
- 即使没被限流，大部分也没启用 `json` format，返回 403
- 公共实例稳定性无法保证

我们对 65 个公共实例进行了批量测试，**无一可用**。所以本地部署是最可靠的方式。

## 总结

| 步骤 | 操作 | 目的 |
|------|------|------|
| 1 | 创建 `~/searxng/settings.yml` | 开启 JSON API，关闭限流 |
| 2 | `docker run` 启动容器 | 部署 SearXNG |
| 3 | `npm install -g mcp-searxng` | 安装 MCP Server |
| 4 | `claude mcp add` 添加 searxng MCP | 让 Claude Code 通过 SearXNG 搜索 |
