# 4.1 核心调度参数

> 返回 [README 总览](../README.md)

调度器决定了如何安排请求的处理顺序和批次大小。这些核心参数直接影响服务的吞吐量和延迟表现。

---

## `--max-num-seqs`

| 属性 | 值 |
|------|-----|
| **说明** | 单次迭代中最大序列数，即同时处理的请求数量上限。值越大，GPU 利用率越高、吞吐量越大，但单个请求的延迟可能增加。值越小，单请求延迟更低，但吞吐量受限 |
| **类型** | int |
| **默认值** | 自动推断 |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --max-num-seqs 64
```
> 允许同时处理最多 64 个序列。适合高并发 API 服务场景，充分利用 GPU 算力提升整体吞吐量。

---

## `--max-num-batched-tokens`

| 属性 | 值 |
|------|-----|
| **说明** | 单次前向传播中处理的最大 token 总数。这个值限制了每次 GPU 计算的工作量上限。支持 `1k`、`2M` 等可读格式。值越大，每次计算处理更多 token，提高吞吐量；值越小，单次计算更快完成，降低延迟 |
| **类型** | int |
| **默认值** | 自动推断 |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --max-num-batched-tokens 8192
```
> 每次前向传播最多处理 8192 个 token，在吞吐量和延迟之间取得平衡，适合中等负载的在线服务场景。

---

## `--scheduling-policy`

| 属性 | 值 |
|------|-----|
| **说明** | 请求调度策略。`fcfs`（First Come First Served）按请求到达顺序处理，公平且简单；`priority` 按优先级调度，允许重要请求优先处理，适合有 VIP 用户分层的业务场景 |
| **类型** | enum |
| **可选值** | `fcfs` / `priority` |
| **默认值** | fcfs |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --scheduling-policy priority
```
> 启用优先级调度策略，配合 API 请求中的 priority 参数，可以实现 VIP 用户请求优先响应，适合有服务等级协议（SLA）分层的生产环境。

---
