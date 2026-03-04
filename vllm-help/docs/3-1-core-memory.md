# 3.1 核心内存参数

> 返回 [README 总览](../README.md)

KV Cache 是 LLM 推理中最关键的内存组件。这些核心参数决定了 GPU 显存的使用比例、KV Cache 的精度以及是否启用前缀缓存优化。

---

## `--gpu-memory-utilization`

| 属性 | 值 |
|------|-----|
| **说明** | GPU 显存利用率，控制 vLLM 最多使用多大比例的 GPU 显存。值越高，可缓存的 KV Cache 越多，能同时服务的请求数越多，但也更容易触发 OOM。这是调节"能同时服务多少请求"的关键旋钮 |
| **类型** | float |
| **默认值** | 0.9 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --gpu-memory-utilization 0.85
```
> 将显存利用率设为 85%，为系统其他进程预留更多显存空间，适合与其他 GPU 任务共享同一张卡的场景。

---

## `--kv-cache-dtype`

| 属性 | 值 |
|------|-----|
| **说明** | KV Cache 的数据类型。`auto` 表示与模型权重精度一致；`fp8` 可将 KV Cache 占用减少约一半，从而支持更多并发或更长上下文，但会有轻微精度损失；`bfloat16` 强制使用 bfloat16 精度存储 |
| **类型** | enum |
| **可选值** | `auto` / `fp8` / `bfloat16` |
| **默认值** | auto |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --kv-cache-dtype fp8
```
> 使用 fp8 精度存储 KV Cache，在几乎不影响生成质量的前提下，显著减少显存占用，可支持更多并发请求或更长的上下文长度。

---

## `--enable-prefix-caching`

| 属性 | 值 |
|------|-----|
| **说明** | 启用前缀缓存（Automatic Prefix Caching）。当多个请求共享相同的 system prompt 或前缀时，vLLM 会自动复用已计算的 KV Cache，避免重复计算，大幅提升吞吐量。在 V1 引擎中默认开启 |
| **类型** | bool |
| **默认值** | None（V1 默认开启） |
| **暴露建议** | 平台默认，用户可改 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --enable-prefix-caching
```
> 显式开启前缀缓存。在客服、助手类应用中，所有请求通常共享相同的系统提示词，启用此选项后首次请求会正常计算，后续相同前缀的请求将直接复用缓存，响应速度显著提升。

---
