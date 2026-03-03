# 4.3 其他调度选项

> 返回 [README 总览](../README.md)

调度器的其他配置选项，包括流式输出控制、异步调度、投机解码支持等。

---

## `--stream-interval`

| 属性 | 值 |
|------|-----|
| **说明** | 流式输出的 token 缓冲大小。控制每次向客户端推送前累积多少个 token。设为 1 表示逐 token 发送，用户体验最流畅但网络开销最大；设为更大的值可以减少 SSE 事件数量从而降低网络开销，但用户感知到的响应会有轻微延迟 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --stream-interval 3
```
> 每累积 3 个 token 后发送一次流式响应，减少约 2/3 的 SSE 事件数量，适合网络带宽有限或客户端连接数极多的场景。

---

## `--async-scheduling`

| 属性 | 值 |
|------|-----|
| **说明** | 启用异步调度。在 GPU 执行当前 batch 的前向传播时，CPU 同时进行下一个 batch 的调度决策，减少 GPU 空闲等待时间，从而提升整体延迟和吞吐量表现 |
| **类型** | bool |
| **默认值** | False |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --async-scheduling
```
> 启用异步调度模式，让调度决策与 GPU 计算并行进行，消除 CPU 调度环节带来的 GPU 空闲间隙，适合高吞吐量生产环境。

---

## `--num-lookahead-slots`

| 属性 | 值 |
|------|-----|
| **说明** | 投机解码（Speculative Decoding）预留的 slot 数量。投机解码使用小模型提前预测多个 token，再由大模型验证。此参数决定了每步预留多少个位置用于投机预测。设为 0 表示不使用投机解码 |
| **类型** | int |
| **默认值** | 0 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --num-lookahead-slots 5 --speculative-config '{"method": "draft_model", "draft_model": "meta-llama/Llama-3.1-8B-Instruct"}'
```
> 预留 5 个 slot 用于投机解码，每次由小模型（8B）预测 5 个 token，再由大模型（70B）一次性验证，正确的 token 直接接受，从而加速生成过程。

---

## `--scheduler-cls`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义调度器类的完整类路径。允许用户使用自己实现的调度算法替代 vLLM 默认调度器，适合需要特殊调度逻辑（如按租户隔离、按模型能力分流等）的高级场景 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --scheduler-cls "my_package.custom_scheduler.TenantAwareScheduler"
```
> 使用自定义的租户感知调度器，按照租户 ID 进行请求隔离和资源分配，适合多租户 SaaS 平台的部署场景。

---

## `--disable-hybrid-kv-cache-manager`

| 属性 | 值 |
|------|-----|
| **说明** | 禁用混合 KV Cache 管理器。默认情况下 vLLM 会根据不同 attention 层的特性（如 GQA、MQA 等）分配不同大小的 KV Cache。启用此选项后，所有 attention 层将分配相同大小的 KV Cache，简化内存管理但可能浪费显存 |
| **类型** | bool |
| **默认值** | False |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --disable-hybrid-kv-cache-manager
```
> 禁用混合 KV Cache 管理，强制所有层使用统一大小的 KV Cache 块。在某些模型或调试场景下可以简化内存分配行为，排查显存相关问题。

---
