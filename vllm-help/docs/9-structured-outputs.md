# 9 结构化输出配置（StructuredOutputsConfig）

> 返回 [README 总览](../README.md)

控制模型按照特定格式（如 JSON Schema）生成输出。结构化输出对需要可靠解析模型输出的应用非常重要，如 Function Calling 和 JSON mode。

---

## `--reasoning-parser`

| 属性 | 值 |
|------|-----|
| **说明** | Reasoning 内容解析器名称，用于将模型的思维链（Chain-of-Thought）输出解析为 OpenAI API 兼容的格式。对于支持推理的模型（如 DeepSeek-R1），此参数可以将思考过程与最终答案分离 |
| **类型** | string |
| **默认值** | 空 |
| **暴露建议** | 平台默认，用户可改 |

**示例：**
```bash
vllm serve deepseek-ai/DeepSeek-R1 --reasoning-parser deepseek_r1
```
> 为 DeepSeek-R1 模型配置专用的推理解析器。解析器会将模型的 `<think>...</think>` 思考过程与最终回答分开，通过 API 的 `reasoning_content` 字段返回思维链内容。

---

## `--reasoning-parser-plugin`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义 reasoning 解析器插件的 Python 模块路径。当内置解析器不满足需求时，可以通过此参数加载自定义的解析器实现 |
| **类型** | string |
| **默认值** | 空 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve my-model --reasoning-parser custom_parser --reasoning-parser-plugin my_plugins.reasoning
```
> 加载 `my_plugins.reasoning` 模块中注册的自定义 reasoning 解析器 `custom_parser`。适用于模型使用非标准的思维链格式时需要自定义解析逻辑的场景。

---

## `--guided-decoding-backend` [已废弃]

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] Guided decoding（引导解码）的后端引擎。用于控制结构化输出（如 JSON Schema）的解码实现。此参数已废弃，结构化输出功能已内置到引擎中，无需手动指定后端 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --guided-decoding-backend xgrammar
```
> 使用 xgrammar 作为引导解码后端。注意此参数已废弃，不建议在新项目中使用。

---

## `--guided-decoding-disable-additional-properties` [已废弃]

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] 在 JSON Schema 引导解码中禁用额外属性。启用后，模型生成的 JSON 将严格遵循 schema 定义，不允许出现未定义的字段。此参数已废弃，相关行为已通过 JSON Schema 本身控制 |
| **类型** | bool |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --guided-decoding-disable-additional-properties
```
> 禁止模型在结构化输出中生成 schema 未定义的额外字段。注意此参数已废弃，不建议在新项目中使用。

---

## `--guided-decoding-disable-any-whitespace` [已废弃]

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] 在引导解码中禁用任意空白字符。默认情况下，结构化输出允许在 JSON 的键值之间插入空白以提高可读性。启用此选项后将生成紧凑的无多余空白的输出。此参数已废弃 |
| **类型** | bool |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --guided-decoding-disable-any-whitespace
```
> 生成紧凑的 JSON 输出，去除不必要的空白字符。注意此参数已废弃，不建议在新项目中使用。

---

## `--guided-decoding-disable-fallback` [已废弃]

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] 禁用引导解码的 fallback 机制。正常情况下，当引导解码失败时会回退到无约束生成。启用此选项后，引导解码失败将直接报错而非静默回退。此参数已废弃 |
| **类型** | bool |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --guided-decoding-disable-fallback
```
> 禁用引导解码的 fallback 机制。当结构化输出约束无法满足时直接返回错误，而不是静默回退到无约束生成。注意此参数已废弃，不建议在新项目中使用。
