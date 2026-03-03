# 5.5 Dual Batch Overlap (DBO)

> 返回 [README 总览](../README.md)

Dual Batch Overlap（DBO）是一种高级调度优化，通过将 prefill 和 decode 阶段重叠执行来提升 GPU 利用率和整体吞吐量。

---

## `--enable-dbo`

| 属性 | 值 |
|------|-----|
| **说明** | 启用双 batch 重叠执行。在 LLM 推理中，prefill（处理输入 prompt）阶段是计算密集型，decode（逐 token 生成）阶段是内存带宽密集型。DBO 将这两个阶段的 batch 在同一 GPU 上交替重叠执行，使 GPU 的计算单元和内存带宽同时被充分利用，从而提升整体吞吐量 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --enable-dbo
```
> 启用 Dual Batch Overlap 优化，在 prefill 和 decode 阶段之间重叠执行，提升 GPU 利用率。

---

## `--dbo-prefill-token-threshold`

| 属性 | 值 |
|------|-----|
| **说明** | Prefill batch 的 token 数量阈值。当待处理的 prefill token 数量达到此阈值时，才会触发一次 prefill batch 的执行。较大的值可以积累更多 token 后一次性处理，提升计算效率；较小的值则响应更及时，适合延迟敏感场景 |
| **类型** | int |
| **默认值** | 512 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --enable-dbo --dbo-prefill-token-threshold 1024
```
> 将 prefill token 阈值设为 1024，积累更多 token 后再执行 prefill，适合长 prompt 高吞吐场景。

---

## `--dbo-decode-token-threshold`

| 属性 | 值 |
|------|-----|
| **说明** | Decode batch 的 token 数量阈值。当待处理的 decode token 数量达到此阈值时，才会触发一次 decode batch 的执行。此值通常远小于 prefill 阈值，因为 decode 阶段每次只生成一个 token，需要更频繁地执行以保持输出流畅性 |
| **类型** | int |
| **默认值** | 32 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --enable-dbo --dbo-decode-token-threshold 64
```
> 将 decode token 阈值设为 64，积累更多并发 decode 请求后批量执行，在高并发场景下提升吞吐量。

---
