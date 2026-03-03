# 1.4 任务与转换

> 返回 [README 总览](../README.md)

定义模型的任务类型（文本生成、嵌入、分类等）以及模型适配器转换。

---

## `--task`

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] 模型任务类型。指定模型执行的任务种类：`generate` 用于文本生成（对话、补全等）；`embed` 用于生成文本向量嵌入；`classify` 用于文本分类；`reward` 用于奖励模型评分；`score` 用于文本相似度评分。此参数已被废弃，vLLM 会自动检测模型的任务类型 |
| **类型** | enum |
| **可选值** | `generate` / `embed` / `classify` / `reward` / `score` |
| **默认值** | None |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve --model BAAI/bge-large-en-v1.5 --task embed
```
> 将模型指定为嵌入任务，用于生成文本的向量表示。

---

## `--convert`

| 属性 | 值 |
|------|-----|
| **说明** | 模型适配器转换。用于将一种类型的模型适配为另一种任务，例如将文本生成模型转换为 pooling 任务使用。`auto` 模式下由引擎自动决定是否需要转换 |
| **类型** | string |
| **默认值** | `auto` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --convert pooling
```
> 将 Llama 文本生成模型适配为 pooling 模式，使其能够用于生成文本嵌入。

---
