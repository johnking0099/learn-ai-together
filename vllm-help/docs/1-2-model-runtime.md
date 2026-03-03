# 1.2 模型运行参数

> 返回 [README 总览](../README.md)

控制模型在 GPU 上的运行方式，包括精度、上下文长度、量化等影响性能和质量的关键参数。

---

## `--max-model-len`

| 属性 | 值 |
|------|-----|
| **说明** | 模型上下文长度（prompt + output 的总 token 数）。这是用户最关心的参数之一：它决定了模型能处理多长的输入+输出。支持 `1k`/`2M` 等可读格式。设置过大会增加显存消耗，设置过小则限制模型的处理能力 |
| **类型** | int |
| **默认值** | 自动从模型配置推导 |
| **暴露建议** | 普通用户 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --max-model-len 8192
```
> 将模型的最大上下文长度设置为 8192 个 token。更长的上下文需要更多显存。

---

## `--dtype`

| 属性 | 值 |
|------|-----|
| **说明** | 模型权重和激活值的数据类型。影响显存占用和推理精度。`auto` 会根据模型配置自动选择最佳精度；`float16` 和 `bfloat16` 是常用的半精度格式；`float32` 为全精度，显存占用翻倍但精度最高 |
| **类型** | enum |
| **可选值** | `auto` / `float16` / `bfloat16` / `float32` |
| **默认值** | `auto` |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --dtype bfloat16
```
> 使用 bfloat16 半精度加载模型，在精度和显存之间取得平衡。

---

## `--quantization` / `-q`

| 属性 | 值 |
|------|-----|
| **说明** | 模型量化方式。量化可以大幅减少显存占用（50% 甚至更多），但会轻微降低精度。常见方式有 AWQ、GPTQ、FP8 等。不同量化方式在速度和精度之间有不同的权衡 |
| **类型** | string |
| **默认值** | None（不量化） |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B -q awq
```
> 使用 AWQ 量化方式加载模型，显著降低显存占用。

---

## `--seed`

| 属性 | 值 |
|------|-----|
| **说明** | 随机种子，用于保证结果可复现。设置相同的种子后，相同的输入会产生相同的输出。V1 版本默认为 0 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --seed 42
```
> 设置随机种子为 42，确保每次推理结果可复现。

---

## `--enforce-eager`

| 属性 | 值 |
|------|-----|
| **说明** | 是否强制使用 eager 模式（禁用 CUDA Graph）。CUDA Graph 通过预编译计算图来加速推理，但在调试时可能隐藏错误。启用 eager 模式后推理速度会降低，但更便于调试 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --enforce-eager
```
> 强制使用 eager 模式运行，禁用 CUDA Graph 优化，适用于调试场景。

---

## `--trust-remote-code`

| 属性 | 值 |
|------|-----|
| **说明** | 是否信任远程代码。某些模型需要执行 HuggingFace 上的自定义代码才能加载（如自定义 attention 实现）。启用此选项有安全隐患，平台应根据模型来源策略统一决定 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model THUDM/chatglm3-6b --trust-remote-code
```
> 允许执行模型仓库中的自定义代码，某些模型（如 ChatGLM）需要此选项才能正常加载。

---

## `--model-impl`

| 属性 | 值 |
|------|-----|
| **说明** | 模型实现方式。`auto` 自动选择最佳实现；`vllm` 使用 vLLM 原生实现（性能最优）；`transformers` 使用 HuggingFace Transformers 实现（兼容性更好）；`terratorch` 使用 TerraTorch 实现 |
| **类型** | enum |
| **可选值** | `auto` / `vllm` / `transformers` / `terratorch` |
| **默认值** | `auto` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --model-impl transformers
```
> 强制使用 HuggingFace Transformers 后端加载模型，适用于 vLLM 原生不支持的模型。

---

## `--runner`

| 属性 | 值 |
|------|-----|
| **说明** | 模型 runner 类型。`auto` 自动选择；`generate` 用于文本生成任务；`pooling` 用于嵌入/分类等池化任务；`draft` 用于投机解码中的草稿模型 |
| **类型** | enum |
| **可选值** | `auto` / `generate` / `pooling` / `draft` |
| **默认值** | `auto` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model BAAI/bge-large-en-v1.5 --runner pooling
```
> 以 pooling 模式运行嵌入模型，用于生成文本向量表示。

---

## `--config-format`

| 属性 | 值 |
|------|-----|
| **说明** | 模型配置格式。`auto` 自动检测；`hf` 使用标准 HuggingFace 配置格式；`mistral` 使用 Mistral 专用配置格式 |
| **类型** | enum |
| **可选值** | `auto` / `hf` / `mistral` |
| **默认值** | `auto` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model mistralai/Mistral-7B-Instruct-v0.3 --config-format mistral
```
> 使用 Mistral 专用配置格式加载模型。

---
