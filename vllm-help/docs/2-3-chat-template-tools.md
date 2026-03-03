# 2.3 Chat Template 与工具调用

> 返回 [README 总览](../README.md)

配置模型的对话模板和 Function Calling（工具调用）能力。Chat Template 决定了多轮对话如何格式化，工具调用让模型能够调用外部函数。

---

## `--chat-template`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义 chat template 的文件路径或 Jinja2 模板字符串。Chat template 控制模型如何理解和格式化多轮对话（system/user/assistant 角色的拼接方式）。大多数模型自带 chat template，仅在需要自定义对话格式时才需要设置 |
| **类型** | string |
| **默认值** | None（使用模型自带） |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --chat-template /path/to/my_template.jinja
```
> 加载自定义的 Jinja2 对话模板文件。适用于使用非标准对话格式训练的模型，或需要定制系统提示词格式的场景。

---

## `--chat-template-content-format`

| 属性 | 值 |
|------|-----|
| **说明** | 控制消息内容在 chat template 中的渲染格式。`auto` 根据模板自动检测；`string` 将内容作为纯字符串传入；`openai` 保持 OpenAI 消息格式的列表结构（支持多模态内容块） |
| **类型** | enum |
| **可选值** | `auto` / `string` / `openai` |
| **默认值** | `auto` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --chat-template-content-format openai
```
> 强制使用 OpenAI 格式渲染消息内容。当模型的 chat template 需要处理包含图片、文本混合的多模态消息时，应使用 `openai` 格式。

---

## `--enable-auto-tool-choice`

| 属性 | 值 |
|------|-----|
| **说明** | 启用模型自动选择工具的能力（Function Calling）。开启后，模型可以根据用户请求自动决定是否调用工具以及调用哪个工具。需要配合 `--tool-call-parser` 指定解析器 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --enable-auto-tool-choice --tool-call-parser hermes
```
> 启用 Function Calling 功能，并使用 Hermes 格式解析工具调用。用户可以在 API 请求中定义工具（functions），模型会自动判断何时调用。

---

## `--tool-call-parser`

| 属性 | 值 |
|------|-----|
| **说明** | 指定工具调用的解析器。不同模型使用不同的工具调用格式，需要匹配对应的解析器。常见解析器包括 `hermes`（通用）、`llama3_json`（Llama 3）、`mistral`（Mistral）、`qwen3_coder`（Qwen3）等 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --enable-auto-tool-choice --tool-call-parser llama3_json
```
> 使用 Llama 3 专用的 JSON 格式工具调用解析器。部署 Llama 3 系列模型并需要 Function Calling 时使用此配置。

---

## `--tool-parser-plugin`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义工具调用解析器的插件文件路径。当内置解析器不满足需求时，可以编写自定义解析器插件并通过此参数加载 |
| **类型** | string |
| **默认值** | 空 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --enable-auto-tool-choice --tool-call-parser custom --tool-parser-plugin /path/to/my_parser.py
```
> 加载自定义的工具调用解析器插件。适用于使用自研或非主流工具调用格式的模型。

---

## `--tool-server`

| 属性 | 值 |
|------|-----|
| **说明** | 外部工具服务器的地址列表（格式为 host:port）。vLLM 可以连接外部工具服务器，自动获取可用的工具定义，使模型能够调用这些远程工具 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --enable-auto-tool-choice --tool-call-parser hermes --tool-server "localhost:9090"
```
> 连接运行在本地 9090 端口的工具服务器。模型在推理时可以自动发现并调用该服务器上注册的工具。

---

## `--exclude-tools-when-tool-choice-none`

| 属性 | 值 |
|------|-----|
| **说明** | 当 API 请求中 `tool_choice` 设为 `none` 时，是否从发送给模型的 prompt 中排除工具定义。启用后可以减少 token 消耗，避免模型在不需要工具时仍受到工具定义的干扰 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --enable-auto-tool-choice --tool-call-parser hermes --exclude-tools-when-tool-choice-none
```
> 当用户请求明确指定不使用工具时，自动从 prompt 中移除工具定义，减少不必要的 token 开销。

---

## `--trust-request-chat-template`

| 属性 | 值 |
|------|-----|
| **说明** | 是否信任并使用 API 请求中携带的自定义 chat template。启用后，客户端可以在请求中动态指定 chat template。此选项存在安全风险，恶意用户可能通过注入恶意模板执行不安全操作 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --trust-request-chat-template
```
> 允许客户端在 API 请求中指定自定义 chat template。仅在受信任的内部环境中使用，公网环境应保持禁用。

---
