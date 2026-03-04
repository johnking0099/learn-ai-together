# 4.2 Chunked Prefill（分块预填充）

> 返回 [README 总览](../README.md)

Chunked Prefill 是 vLLM 的重要性能特性：将长 prompt 的预填充分成多个小块，与正在生成的请求交错执行，从而减少长 prompt 导致的延迟峰值。

---

## `--enable-chunked-prefill`

| 属性 | 值 |
|------|-----|
| **说明** | 启用分块预填充功能。开启后，长 prompt 的预填充计算会被分成多个小块，与 decode 阶段的请求交错执行。这样可以避免一个超长 prompt 独占 GPU 导致其他正在生成的请求卡顿。在 V1 引擎中默认启用 |
| **类型** | bool |
| **默认值** | None（V1 默认启用） |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --enable-chunked-prefill
```
> 显式启用分块预填充。当服务同时处理新请求（预填充阶段）和正在生成的请求（解码阶段）时，避免长 prompt 独占 GPU 造成已有请求的延迟抖动。

---

## `--max-num-partial-prefills`

| 属性 | 值 |
|------|-----|
| **说明** | 最大并发部分预填充序列数。限制同时进行分块预填充的序列数量，防止过多的预填充任务抢占 decode 阶段的计算资源 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --enable-chunked-prefill --max-num-partial-prefills 2
```
> 允许最多 2 个序列同时进行分块预填充，在新请求的预填充速度和已有请求的解码延迟之间取得更好的平衡。

---

## `--max-long-partial-prefills`

| 属性 | 值 |
|------|-----|
| **说明** | 长 prompt 最大并发预填充数。专门限制被判定为"长 prompt"的序列同时进行预填充的数量，防止多个超长 prompt 同时预填充导致 GPU 过载 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --enable-chunked-prefill --max-long-partial-prefills 1
```
> 限制同一时刻最多只有 1 个长 prompt 在进行预填充，确保长上下文请求不会过度抢占 GPU 资源。

---

## `--long-prefill-token-threshold`

| 属性 | 值 |
|------|-----|
| **说明** | 判定"长 prompt"的 token 阈值。当一个请求的 prompt token 数超过此阈值时，会被归类为"长 prompt"，受 `--max-long-partial-prefills` 的限制。设为 0 表示禁用长短 prompt 区分 |
| **类型** | int |
| **默认值** | 0 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --enable-chunked-prefill --long-prefill-token-threshold 4096 --max-long-partial-prefills 1
```
> 将超过 4096 token 的 prompt 视为长 prompt，并限制同时只有 1 个长 prompt 进行预填充，短 prompt 则不受此限制，实现差异化调度。

---

## `--disable-chunked-mm-input`

| 属性 | 值 |
|------|-----|
| **说明** | 禁止将多模态输入（如图片、视频对应的 token）进行分块调度。某些多模态模型的编码器要求输入完整性，分块可能导致处理异常，此时需要禁用多模态输入的分块 |
| **类型** | bool |
| **默认值** | False |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve llava-hf/llava-v1.6-mistral-7b-hf --enable-chunked-prefill --disable-chunked-mm-input
```
> 在多模态模型中启用分块预填充的同时，保持多模态输入（图片 token）的完整性不被分块，避免视觉编码器处理不完整输入导致的质量下降。

---
