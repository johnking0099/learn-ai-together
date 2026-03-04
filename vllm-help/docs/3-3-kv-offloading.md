# 3.3 KV Offloading

> 返回 [README 总览](../README.md)

KV Offloading 允许将部分 KV Cache 从 GPU 显存卸载到其他存储介质，从而在有限的 GPU 显存下支持更大的上下文长度或更多并发请求。

---

## `--kv-offloading-backend`

| 属性 | 值 |
|------|-----|
| **说明** | KV Cache 卸载后端的选择。`native` 使用 vLLM 原生的 CPU 卸载实现，将部分 KV Cache 从 GPU 转移到 CPU 内存；`lmcache` 使用 LMCache 作为外部缓存后端，支持更灵活的缓存策略和分布式缓存 |
| **类型** | enum |
| **可选值** | `native` / `lmcache` |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --kv-offloading-backend native --kv-offloading-size 4
```
> 使用 vLLM 原生的 KV 卸载后端，将部分 KV Cache 卸载到 CPU 内存，在 GPU 显存有限时有效扩展可支持的上下文长度。

---

## `--kv-offloading-size`

| 属性 | 值 |
|------|-----|
| **说明** | KV Cache 卸载缓冲区的大小（GiB）。决定了在 CPU 端或外部缓存中为 KV Cache 卸载预留多少存储空间。值越大，可卸载的 KV Cache 越多，从而支持更长的上下文或更多并发请求 |
| **类型** | float |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --kv-offloading-backend native --kv-offloading-size 8
```
> 为 KV Cache 卸载分配 8 GiB 的缓冲区空间，适合需要处理超长上下文（如 128K tokens）的场景，在保持合理延迟的同时避免 GPU 显存溢出。

---
