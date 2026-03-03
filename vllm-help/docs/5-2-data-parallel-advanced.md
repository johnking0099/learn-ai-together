# 5.2 数据并行高级配置

> 返回 [README 总览](../README.md)

数据并行高级配置用于精细控制多副本部署的通信、负载均衡和同步策略。这些参数主要在多节点数据并行场景下使用。

---

## `--data-parallel-size-local` / `-dpl`

| 属性 | 值 |
|------|-----|
| **说明** | 本节点上的数据并行副本数。在多节点数据并行部署中，指定当前物理节点上运行多少个模型副本。如果不设置，默认将所有副本分配到本节点。适用于异构集群中不同节点承载不同数量副本的场景 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --data-parallel-size-local 2 --tensor-parallel-size 2
```
> 总共 4 个数据并行副本，当前节点运行其中 2 个副本，每个副本使用 2 张 GPU 进行张量并行。

---

## `--data-parallel-address` / `-dpa`

| 属性 | 值 |
|------|-----|
| **说明** | 数据并行集群头节点（rank 0）的 IP 地址。在多节点数据并行部署中，所有节点需要知道头节点地址以建立通信。仅在多节点部署时需要设置 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --data-parallel-address 192.168.1.100 --data-parallel-rank 1
```
> 在从节点上启动数据并行实例，连接到头节点 192.168.1.100。

---

## `--data-parallel-backend` / `-dpb`

| 属性 | 值 |
|------|-----|
| **说明** | 数据并行后端选择。`mp` 使用 Python 多进程方式管理多个副本，适合单机场景；`ray` 使用 Ray 分布式框架，适合多机集群场景，提供更强大的资源调度和容错能力 |
| **类型** | enum |
| **可选值** | `mp` / `ray` |
| **默认值** | `mp` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --data-parallel-backend ray
```
> 使用 Ray 作为数据并行后端管理 4 个模型副本，适合在 Ray 集群上进行多节点部署。

---

## `--data-parallel-rank` / `-dpn`

| 属性 | 值 |
|------|-----|
| **说明** | 当前实例在数据并行集群中的 rank 编号（从 0 开始）。在多节点数据并行部署中，每个节点需要分配唯一的 rank 标识，头节点通常为 rank 0 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --data-parallel-address 192.168.1.100 --data-parallel-rank 2
```
> 启动数据并行集群中的第 3 个实例（rank 2），连接到头节点 192.168.1.100。

---

## `--data-parallel-rpc-port` / `-dpp`

| 属性 | 值 |
|------|-----|
| **说明** | 数据并行 RPC 通信端口。用于数据并行副本之间的远程过程调用通信。在多节点部署时需确保此端口在所有节点间可达且未被占用 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --data-parallel-rpc-port 29600
```
> 指定数据并行 RPC 通信使用端口 29600。

---

## `--data-parallel-start-rank` / `-dpr`

| 属性 | 值 |
|------|-----|
| **说明** | 次要节点（非头节点）的起始 rank 编号。用于在从节点上批量启动多个数据并行副本时，指定副本 rank 编号的起始值 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 8 --data-parallel-size-local 4 --data-parallel-start-rank 4 --data-parallel-address 192.168.1.100
```
> 在从节点上启动 4 个副本，rank 从 4 开始（即 rank 4、5、6、7），连接到头节点 192.168.1.100 上的 rank 0-3。

---

## `--data-parallel-external-lb` / `-dpe`

| 属性 | 值 |
|------|-----|
| **说明** | 启用外部负载均衡器模式。开启后，vLLM 不进行内部的请求分发，而是依赖外部负载均衡器（如 Nginx、HAProxy 或 Kubernetes Service）将请求分发到不同的数据并行副本 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --data-parallel-external-lb
```
> 启用外部负载均衡模式，由 Kubernetes Service 或 Nginx 等外部组件负责将请求分发到 4 个副本。

---

## `--data-parallel-hybrid-lb` / `-dph`

| 属性 | 值 |
|------|-----|
| **说明** | 启用混合负载均衡模式。结合 vLLM 内部负载感知调度和外部负载均衡的优点，在保持外部流量入口的同时，利用 vLLM 内部的队列状态信息进行更智能的请求分配 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --data-parallel-hybrid-lb
```
> 启用混合负载均衡模式，在外部负载均衡器基础上利用 vLLM 内部队列信息优化请求分配。

---

## `--disable-nccl-for-dp-synchronization`

| 属性 | 值 |
|------|-----|
| **说明** | 数据并行同步时使用 Gloo 后端替代 NCCL。NCCL 是 NVIDIA 官方的 GPU 集合通信库，性能更好；Gloo 是 CPU 端的通信库，兼容性更强。在某些网络环境或非 NVIDIA GPU 场景下，可能需要切换到 Gloo |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --data-parallel-size 4 --disable-nccl-for-dp-synchronization
```
> 数据并行同步使用 Gloo 后端而非 NCCL，适用于 NCCL 不可用或存在兼容性问题的环境。

---
