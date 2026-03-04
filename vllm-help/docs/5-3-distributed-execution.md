# 5.3 分布式执行

> 返回 [README 总览](../README.md)

分布式执行配置控制多机多卡推理的通信架构，包括分布式后端选择、节点管理和通信地址配置。

---

## `--distributed-executor-backend`

| 属性 | 值 |
|------|-----|
| **说明** | 分布式执行后端选择。`mp` 使用 Python 多进程，适合单机多卡场景，启动快、开销小；`ray` 使用 Ray 分布式框架，适合多机多卡场景，支持跨节点通信和资源调度；`uni` 使用单进程模式，所有计算在一个进程内完成，适合调试；`external_launcher` 使用外部启动器（如 torchrun）管理进程 |
| **类型** | enum |
| **可选值** | `mp` / `ray` / `uni` / `external_launcher` |
| **默认值** | 自动选择 |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --distributed-executor-backend ray
```
> 使用 Ray 作为分布式执行后端管理 4 张 GPU 上的张量并行推理，适合在 Ray 集群环境中运行。

---

## `--nnodes` / `-n`

| 属性 | 值 |
|------|-----|
| **说明** | 多机推理时参与计算的节点总数。当模型需要跨越多台物理机器部署时，需要指定参与的节点数量。例如 2 表示使用两台机器协同推理 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-405B-Instruct --tensor-parallel-size 8 --nnodes 2 --node-rank 0 --master-addr 192.168.1.100
```
> 在 2 台机器上部署 405B 模型，每台机器使用 8 张 GPU，总共 16 张 GPU 进行张量并行。此命令在头节点（rank 0）上执行。

---

## `--node-rank` / `-r`

| 属性 | 值 |
|------|-----|
| **说明** | 当前节点在多机集群中的 rank 编号（从 0 开始）。头节点为 rank 0，其余节点依次递增。每个节点必须有唯一的 rank |
| **类型** | int |
| **默认值** | 0 |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-405B-Instruct --tensor-parallel-size 8 --nnodes 2 --node-rank 1 --master-addr 192.168.1.100
```
> 在从节点（rank 1）上启动分布式推理，连接到头节点 192.168.1.100。

---

## `--master-addr`

| 属性 | 值 |
|------|-----|
| **说明** | 主节点（rank 0）的 IP 地址或主机名。所有节点通过此地址与主节点建立通信连接。在单机场景下使用默认的 127.0.0.1 即可，多机场景下需设置为主节点的实际网络地址 |
| **类型** | string |
| **默认值** | `127.0.0.1` |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-405B-Instruct --tensor-parallel-size 16 --nnodes 2 --master-addr 10.0.0.1 --master-port 29501
```
> 指定主节点地址为 10.0.0.1，所有工作节点将连接到此地址进行分布式通信。

---

## `--master-port`

| 属性 | 值 |
|------|-----|
| **说明** | 主节点的通信端口。用于分布式进程组初始化时的握手通信。需确保此端口在主节点上未被占用且对所有工作节点可达 |
| **类型** | int |
| **默认值** | 29501 |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-405B-Instruct --tensor-parallel-size 16 --nnodes 2 --master-addr 10.0.0.1 --master-port 30000
```
> 使用端口 30000 进行分布式通信，避免与默认端口冲突。

---

## `--max-parallel-loading-workers`

| 属性 | 值 |
|------|-----|
| **说明** | 模型加载时的最大并行 worker 数。控制同时从磁盘读取和加载模型权重的并发数。适当限制可以防止多 GPU 同时加载大模型时发生内存溢出（OOM）。设为 None 表示不限制 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 8 --max-parallel-loading-workers 2
```
> 8 张 GPU 的推理场景下，限制最多 2 个 worker 同时加载模型权重，避免在模型加载阶段因内存压力过大导致 OOM。

---
