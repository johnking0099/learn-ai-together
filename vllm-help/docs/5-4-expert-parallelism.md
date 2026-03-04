# 5.4 Expert Parallelism（专家并行，MoE 模型）

> 返回 [README 总览](../README.md)

Expert Parallelism 专用于 Mixture-of-Experts (MoE) 模型（如 Mixtral、DeepSeek-V3），将不同的专家网络分布在不同 GPU 上，以实现高效的大规模 MoE 推理。

---

## `--enable-expert-parallel`

| 属性 | 值 |
|------|-----|
| **说明** | 对 MoE 层启用专家并行替代张量并行。在 MoE 模型中，每一层包含多个"专家"子网络，常规张量并行会拆分每个专家，而专家并行则将不同的完整专家分配到不同 GPU 上。这对于专家数量远大于 GPU 数量的模型（如 DeepSeek-V3 有 256 个专家）特别有效 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve deepseek-ai/DeepSeek-V3 --tensor-parallel-size 8 --enable-expert-parallel
```
> 部署 DeepSeek-V3 模型时启用专家并行，将 256 个专家分布到 8 张 GPU 上，每张 GPU 负责 32 个专家。

---

## `--expert-placement-strategy`

| 属性 | 值 |
|------|-----|
| **说明** | 专家在 GPU 上的放置策略。`linear` 按顺序连续放置，即将专家 0-31 放在 GPU 0，32-63 放在 GPU 1，依此类推；`round_robin` 按轮询方式放置，即专家 0 放 GPU 0、专家 1 放 GPU 1，循环分配，可实现更均匀的负载分布 |
| **类型** | enum |
| **可选值** | `linear` / `round_robin` |
| **默认值** | `linear` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve mistralai/Mixtral-8x7B-Instruct-v0.1 --tensor-parallel-size 4 --enable-expert-parallel --expert-placement-strategy round_robin
```
> 使用轮询策略将 Mixtral 的 8 个专家均匀分布到 4 张 GPU 上，使每张 GPU 处理的专家负载更均衡。

---

## `--all2all-backend`

| 属性 | 值 |
|------|-----|
| **说明** | MoE 模型中 All-to-All 通信的后端实现。All-to-All 通信是专家并行的核心操作——每个 GPU 需要将 token 发送到持有对应专家的 GPU 上。`naive` 使用基础实现；`pplx` 使用 PPLX 优化库；`deepep_high_throughput` 和 `deepep_low_latency` 分别针对高吞吐和低延迟场景优化，来自 DeepEP 库 |
| **类型** | enum |
| **可选值** | `naive` / `pplx` / `deepep_high_throughput` / `deepep_low_latency` |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve deepseek-ai/DeepSeek-V3 --tensor-parallel-size 8 --enable-expert-parallel --all2all-backend deepep_high_throughput
```
> 使用 DeepEP 高吞吐后端进行专家间的 All-to-All 通信，适合离线批处理等注重吞吐量的场景。

---

## `--enable-eplb`

| 属性 | 值 |
|------|-----|
| **说明** | 启用 Expert Parallelism 负载均衡（EPLB）。在 MoE 模型中，不同专家的激活频率通常不均匀（某些"热门"专家被选中的概率更高），导致 GPU 间负载不均。EPLB 通过动态调整专家分配策略来平衡各 GPU 的计算负载 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve deepseek-ai/DeepSeek-V3 --tensor-parallel-size 8 --enable-expert-parallel --enable-eplb
```
> 启用专家并行负载均衡，自动检测热门专家并重新分配，防止部分 GPU 成为瓶颈。

---

## `--eplb-config`

| 属性 | 值 |
|------|-----|
| **说明** | Expert Parallelism 负载均衡的详细配置，以 JSON 格式传入。可以配置负载统计窗口、重平衡频率、专家复制策略等高级选项。需配合 `--enable-eplb` 使用 |
| **类型** | string (JSON) |
| **默认值** | 默认配置 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve deepseek-ai/DeepSeek-V3 --tensor-parallel-size 8 --enable-expert-parallel --enable-eplb --eplb-config '{"rebalance_interval": 100}'
```
> 启用专家并行负载均衡，并通过 JSON 配置设置每 100 次迭代进行一次专家重平衡。

---
