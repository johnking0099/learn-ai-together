# 5.1 核心并行参数

> 返回 [README 总览](../README.md)

当单张 GPU 无法容纳大模型时，需要多 GPU 甚至多机并行推理。张量并行、流水线并行和数据并行是三种基本的并行策略。

---

## `--tensor-parallel-size` / `-tp`

| 属性 | 值 |
|------|-----|
| **说明** | 张量并行数。将模型的每一层拆分到多张 GPU 上并行计算。例如 `-tp 4` 表示将模型的每一层切分到 4 张 GPU 上，每张 GPU 持有该层 1/4 的权重。这是最常用的多 GPU 推理方式，适合单机多卡场景。TP 数通常应为 GPU 数量的因子，且不超过单机 GPU 总数 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 普通用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4
```
> 使用 4 张 GPU 进行张量并行推理，将 70B 模型的每一层拆分到 4 张 GPU 上，每张 GPU 分担约 17.5B 参数量的计算。

---

## `--pipeline-parallel-size` / `-pp`

| 属性 | 值 |
|------|-----|
| **说明** | 流水线并行数。将模型的不同层分配到不同 GPU 上，形成流水线。例如 `-pp 2` 表示前一半层在 GPU 0，后一半层在 GPU 1。适合显存受限但 GPU 数量多的场景，或跨节点部署时减少通信开销。通常与张量并行组合使用，总 GPU 数 = tp * pp |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 2 --pipeline-parallel-size 2
```
> 使用 4 张 GPU（2 路张量并行 x 2 路流水线并行）。每 2 张 GPU 负责模型一半的层，每层再在 2 张 GPU 间拆分。适合跨节点部署，每个节点 2 张 GPU 的场景。

---

## `--data-parallel-size` / `-dp`

| 属性 | 值 |
|------|-----|
| **说明** | 数据并行数。运行多个完整的模型副本，每个副本独立处理不同的请求，线性扩展吞吐量。例如 `-dp 2 -tp 4` 表示部署 2 个副本，每个副本使用 4 张 GPU，共需 8 张 GPU。数据并行是最简单的扩展吞吐量方式，副本间无需通信 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --data-parallel-size 2
```
> 部署 2 个模型副本，每个副本使用 4 张 GPU 进行张量并行，共使用 8 张 GPU。吞吐量相比单副本提升约 2 倍，适合高并发在线服务场景。

---
