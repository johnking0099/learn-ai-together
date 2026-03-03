# 5.6 Context Parallelism

> 返回 [README 总览](../README.md)

Context Parallelism 将序列的上下文分布到多个设备上进行并行处理，适用于超长上下文场景。

---

## `--decode-context-parallel-size` / `-dcp`

| 属性 | 值 |
|------|-----|
| **说明** | Decode 阶段的 Context 并行度。将序列的 KV Cache 上下文分片到多个设备上，每个设备只负责一部分上下文的 attention 计算。适用于超长上下文推理场景（如 128K、1M tokens），此时单张 GPU 的显存无法容纳完整的 KV Cache。值为 1 时不启用 Context 并行 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --decode-context-parallel-size 2 --max-model-len 128000
```
> 在 128K 上下文长度场景下，使用 2 路 Context 并行，将 KV Cache 分布到 2 组设备上，有效降低单设备显存压力。

---

## `--dcp-kv-cache-interleave-size`

| 属性 | 值 |
|------|-----|
| **说明** | DCP（Decode Context Parallelism）中 KV Cache 交错分片的块大小。控制将 KV Cache 切分到不同设备时每个分片包含的连续 token 数。较大的值减少跨设备通信次数但可能导致负载不均，较小的值实现更均匀的分布但增加通信开销 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --decode-context-parallel-size 2 --dcp-kv-cache-interleave-size 4 --max-model-len 128000
```
> 使用 2 路 Context 并行，KV Cache 以每 4 个 token 为一组交错分配到不同设备上，平衡通信效率和负载均匀性。

---
