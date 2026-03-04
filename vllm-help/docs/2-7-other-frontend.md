# 2.7 其他前端选项

> 返回 [README 总览](../README.md)

前端服务的其他配置选项，包括响应格式、API 文档、使用量跟踪、中间件等。这些选项大多面向平台管理员。

---

## `--response-role`

| 属性 | 值 |
|------|-----|
| **说明** | 当 `add_generation_prompt=true` 时，设置模型响应消息中的角色名称。在兼容 OpenAI API 的响应格式中，此参数决定了生成内容所对应的角色标识 |
| **类型** | string |
| **默认值** | `assistant` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --response-role bot
```
> 将模型响应的角色名从默认的 `assistant` 改为 `bot`。适用于自定义对话系统中需要特定角色标识的场景。

---

## `--disable-frontend-multiprocessing`

| 属性 | 值 |
|------|-----|
| **说明** | 禁用前端多进程模式，将前端 API 服务和推理引擎运行在同一个进程中。默认情况下 vLLM 使用独立进程运行前端和引擎以获得更好的性能，但在调试或资源受限时可以禁用 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --disable-frontend-multiprocessing
```
> 在单进程中同时运行 API 前端和推理引擎。适用于调试场景或资源受限的开发环境。

---

## `--disable-fastapi-docs`

| 属性 | 值 |
|------|-----|
| **说明** | 禁用 FastAPI 自动生成的 Swagger UI 交互式 API 文档（默认位于 `/docs` 路径）。在生产环境中关闭可以减少攻击面，避免暴露 API 接口详情 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --disable-fastapi-docs
```
> 关闭 Swagger UI 文档页面。生产环境中建议禁用，避免未授权用户通过 `/docs` 页面了解 API 结构。

---

## `--enable-force-include-usage`

| 属性 | 值 |
|------|-----|
| **说明** | 强制在每次 API 响应中都包含 `usage` 信息（token 用量统计）。默认情况下，流式响应可能不包含 usage 信息，启用后无论流式还是非流式响应都会返回 prompt_tokens 和 completion_tokens |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --enable-force-include-usage
```
> 确保所有响应（包括流式）都包含 token 用量信息。适用于需要精确计费或监控 token 使用量的平台。

---

## `--enable-prompt-tokens-details`

| 属性 | 值 |
|------|-----|
| **说明** | 在响应的 `usage` 字段中包含 `prompt_tokens_details`，提供更详细的 prompt token 统计信息（如缓存命中的 token 数量等） |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --enable-prompt-tokens-details --enable-force-include-usage
```
> 在 usage 信息中增加 prompt token 的详细分类统计。配合 prefix caching 使用时，可以查看有多少 token 命中了缓存。

---

## `--enable-request-id-headers`

| 属性 | 值 |
|------|-----|
| **说明** | 在 API 响应的 HTTP Header 中添加 `X-Request-Id` 字段。每个请求会分配一个唯一 ID，方便在分布式系统中进行请求追踪和日志关联 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --enable-request-id-headers
```
> 在响应头中添加请求 ID。客户端可以通过 `X-Request-Id` 头跟踪请求，排查特定请求的问题时将此 ID 提供给运维团队。

---

## `--enable-server-load-tracking`

| 属性 | 值 |
|------|-----|
| **说明** | 启用服务器负载指标跟踪。开启后可以监控服务器当前的请求处理负载情况，为负载均衡和自动扩缩容提供数据依据 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --enable-server-load-tracking
```
> 启用负载指标收集。平台可以基于这些指标实现智能路由，将新请求分配到负载较低的实例上。

---

## `--enable-tokenizer-info-endpoint`

| 属性 | 值 |
|------|-----|
| **说明** | 启用 `/get_tokenizer_info` API 端点。该端点返回当前加载模型的 tokenizer 信息，包括词表大小、特殊 token 等。对于需要在客户端进行 token 计数的应用很有用 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --enable-tokenizer-info-endpoint
```
> 开放 tokenizer 信息查询接口。客户端可以通过 `GET /get_tokenizer_info` 获取 tokenizer 详情，用于本地 token 计数或 prompt 截断。

---

## `--return-tokens-as-token-ids`

| 属性 | 值 |
|------|-----|
| **说明** | 将响应中的 token 以 `token_id:{id}` 格式返回，而非原始 token 文本。适用于需要直接处理 token ID 的下游应用 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --return-tokens-as-token-ids
```
> 在 logprobs 等字段中返回 token ID 而非文本。适用于需要进行 token 级别分析或与其他模型进行 token 对齐的场景。

---

## `--middleware`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义 ASGI 中间件的模块路径列表。中间件可以在请求处理前后执行额外逻辑，如认证、限流、请求日志、请求改写等 |
| **类型** | string |
| **默认值** | `[]` |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve --middleware "myapp.middleware.AuthMiddleware" --middleware "myapp.middleware.RateLimitMiddleware"
```
> 加载自定义的认证和限流中间件。平台可以通过中间件实现自定义的请求鉴权、流量控制和访问审计等功能。

---

## `--tokens-only`

| 属性 | 值 |
|------|-----|
| **说明** | 仅启用 Tokens In/Out 端点，用于 Disaggregated（分离式）部署架构。在此模式下，服务仅处理 token 的输入和输出，不提供完整的 Chat/Completion API |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --tokens-only
```
> 以 token 专用模式启动服务。适用于 Prefill-Decode 分离部署架构中，作为专门的 token 处理节点。

---

## `--h11-max-header-count`

| 属性 | 值 |
|------|-----|
| **说明** | 设置 HTTP 请求允许的最大 Header 数量（h11 HTTP 解析器参数）。当客户端发送的请求头数量超过此限制时，请求会被拒绝 |
| **类型** | int |
| **默认值** | 256 |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve --h11-max-header-count 512
```
> 将允许的最大 HTTP Header 数量增加到 512。当客户端需要发送大量自定义头部信息时可能需要调整此值。

---

## `--h11-max-incomplete-event-size`

| 属性 | 值 |
|------|-----|
| **说明** | 设置不完整 HTTP 事件的最大缓冲大小（字节）。h11 HTTP 解析器在接收不完整的 HTTP 请求时会进行缓冲，此参数限制缓冲区的最大大小，防止恶意的超大请求消耗过多内存 |
| **类型** | int |
| **默认值** | 4194304 (4MB) |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve --h11-max-incomplete-event-size 8388608
```
> 将不完整 HTTP 事件的最大缓冲大小增加到 8MB。当合法请求的 Header 或 Body 特别大时（如携带大量工具定义），可能需要适当增大此值。

---
