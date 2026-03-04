# 1.1 模型选择与标识

> 返回 [README 总览](../README.md)

这些参数决定了"部署哪个模型"以及模型的标识信息。`--model` 是最核心的参数，用户通过它指定想要部署的 HuggingFace 模型或本地模型路径。

---

## model_tag (位置参数)

| 属性 | 值 |
|------|-----|
| **说明** | 要部署的模型标签，也可通过 config 文件指定。这是一个位置参数，可以直接写在 `vllm serve` 命令之后，无需加 `--` 前缀 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 基础配置 |

**示例：**
```bash
vllm serve Qwen/Qwen3-0.6B
```
> 直接将模型标签作为位置参数传入，等效于 `--model Qwen/Qwen3-0.6B`。

---

## `--model`

| 属性 | 值 |
|------|-----|
| **说明** | 模型的 HuggingFace 名称或本地路径。这是最核心的参数——用户通过它指定想部署哪个模型。可以是 HuggingFace Hub 上的模型 ID（如 `meta-llama/Llama-3.1-70B-Instruct`），也可以是本地文件系统上的模型路径 |
| **类型** | string |
| **默认值** | `Qwen/Qwen3-0.6B` |
| **暴露建议** | 基础配置 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct
```
> 从 HuggingFace Hub 加载 Llama 3.1 70B Instruct 模型进行部署。

---

## `--served-model-name`

| 属性 | 值 |
|------|-----|
| **说明** | API 中暴露的模型名称。用户调用 API 时用此名称引用模型。支持设置多个别名，方便业务区分不同部署 |
| **类型** | string |
| **默认值** | 与 `--model` 相同 |
| **暴露建议** | 基础配置 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --served-model-name my-llama
```
> 将模型在 API 中命名为 `my-llama`，客户端调用时使用 `model: "my-llama"` 即可。

---

## `--revision`

| 属性 | 值 |
|------|-----|
| **说明** | 模型的具体版本（branch/tag/commit id）。用于锁定特定版本避免意外更新，确保生产环境中模型行为的一致性 |
| **类型** | string |
| **默认值** | None（使用默认版本） |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --revision v1.0
```
> 指定加载 Qwen3-0.6B 的 `v1.0` 版本，避免因模型更新导致行为变化。

---

## `--tokenizer`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义 tokenizer 的名称或路径。大多数情况下无需设置，使用模型自带的即可。仅在需要替换默认 tokenizer 时使用 |
| **类型** | string |
| **默认值** | 与 model 相同 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --tokenizer /path/to/custom-tokenizer
```
> 使用本地自定义的 tokenizer 替代模型自带的 tokenizer。

---

## `--tokenizer-revision`

| 属性 | 值 |
|------|-----|
| **说明** | Tokenizer 的版本号。与 `--revision` 类似，但专门用于锁定 tokenizer 的版本 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --tokenizer-revision abc123
```
> 锁定 tokenizer 到指定的 commit hash `abc123`。

---

## `--tokenizer-mode`

| 属性 | 值 |
|------|-----|
| **说明** | Tokenizer 模式。`auto` 会自动选择最优实现；`slow` 使用 Python 实现（兼容性更好但更慢）；`mistral` 专用于 Mistral 模型的 tokenizer；`custom` 使用自定义实现 |
| **类型** | enum |
| **可选值** | `auto` / `slow` / `mistral` / `custom` |
| **默认值** | `auto` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --model mistralai/Mistral-7B-Instruct-v0.3 --tokenizer-mode mistral
```
> 使用 Mistral 专用的 tokenizer 模式加载模型。

---

## `--hf-token`

| 属性 | 值 |
|------|-----|
| **说明** | HuggingFace 访问令牌，用于下载需要授权的模型（如 Llama 系列）。涉及凭证安全，应由平台统一管理（如通过 Secret 注入），不应让用户在 UI 中输入 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 基础配置 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --hf-token hf_xxxxxxxxxxxxxxxxxxxx
```
> 使用 HuggingFace 令牌下载需要授权访问的 Llama 3.1 模型。

---
