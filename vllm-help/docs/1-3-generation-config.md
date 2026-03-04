# 1.3 生成配置

> 返回 [README 总览](../README.md)

控制模型生成文本时的默认行为，如温度、logprobs 等。这些参数设定引擎级别的默认值，可被 API 请求级别的参数覆盖。

---

## `--generation-config`

| 属性 | 值 |
|------|-----|
| **说明** | 生成配置来源。`auto` 从模型目录中的 `generation_config.json` 加载；`vllm` 使用 vLLM 内置的默认值；也可以指定一个文件夹路径，从该路径下的 `generation_config.json` 读取配置 |
| **类型** | string |
| **默认值** | `auto` |
| **暴露建议** | 平台默认，用户可改 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --generation-config vllm
```
> 使用 vLLM 内置默认生成配置，而非模型自带的 `generation_config.json`。

---

## `--override-generation-config`

| 属性 | 值 |
|------|-----|
| **说明** | 覆盖生成配置的 JSON 字符串。可以设置全局默认的生成参数，如温度（temperature）、top_p 等。这些默认值可被单个 API 请求中的参数覆盖 |
| **类型** | string |
| **默认值** | `{}` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --override-generation-config '{"temperature": 0.7, "top_p": 0.9}'
```
> 设置全局默认的生成温度为 0.7、top_p 为 0.9。单次 API 请求仍可覆盖这些值。

---

## `--max-logprobs`

| 属性 | 值 |
|------|-----|
| **说明** | 返回的最大 log probabilities 数量。限制 API 请求中 `logprobs` 参数的上限。设置为 -1 表示不限制，但可能导致 OOM（内存不足） |
| **类型** | int |
| **默认值** | 20 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --max-logprobs 50
```
> 允许 API 请求最多返回 50 个 token 的 log probabilities。

---

## `--logprobs-mode`

| 属性 | 值 |
|------|-----|
| **说明** | Logprobs 内容类型。`raw_logprobs` 返回原始的对数概率值；`processed_logprobs` 返回经过处理（如应用温度、top_p 等采样参数）后的对数概率值 |
| **类型** | enum |
| **可选值** | `raw_logprobs` / `processed_logprobs` |
| **默认值** | `raw_logprobs` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --logprobs-mode processed_logprobs
```
> 返回经过采样参数处理后的 logprobs，而非原始值。

---

## `--logits-processor-pattern`

| 属性 | 值 |
|------|-----|
| **说明** | 允许的 logits processor 正则匹配模式。用于限制 API 请求中可以使用的自定义 logits processor，防止加载不受信任的代码。未设置时不允许任何自定义 processor |
| **类型** | string |
| **默认值** | None（不允许） |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --logits-processor-pattern "my_module\\..*"
```
> 仅允许匹配 `my_module.*` 模式的 logits processor 被 API 请求调用。

---

## `--logits-processors`

| 属性 | 值 |
|------|-----|
| **说明** | 预加载的 logits processors 列表。在引擎启动时加载指定的 logits processor，供所有请求使用。这些 processor 可以在生成过程中修改 token 的概率分布 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --logits-processors my_module.CustomProcessor
```
> 预加载自定义的 logits processor `my_module.CustomProcessor`，在每次生成时自动应用。

---
