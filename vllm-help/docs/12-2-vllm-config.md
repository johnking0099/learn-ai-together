# 12.2 VllmConfig 顶层配置

> 返回 [README 总览](../README.md)

VllmConfig 是 vLLM 的顶层配置容器，包含一些全局性的高级功能配置，如投机解码、分布式 KV Cache 传输等。

---

## `--additional-config`

| 属性 | 值 |
|------|-----|
| **说明** | 特定平台的额外配置参数，以 JSON 字典形式传入。用于传递 vLLM 核心参数体系之外的自定义配置，通常由平台或插件系统使用。不同的部署平台或扩展模块可以通过此参数接收自己的配置信息 |
| **类型** | string (JSON dict) |
| **默认值** | `{}` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen3-0.6B --additional-config '{"custom_platform_key": "value"}'
```
> 传递平台特定的扩展配置参数，供自定义插件或平台适配层使用。

---

## `--speculative-config`

| 属性 | 值 |
|------|-----|
| **说明** | 投机解码（Speculative Decoding）配置。投机解码使用一个小而快的 draft model 来预测大模型的输出 token，然后由大模型一次性验证多个 token，从而加速推理过程。配置项包括 draft model 路径、预测 token 数量等，以 JSON 字典形式传入 |
| **类型** | string (JSON dict) |
| **默认值** | None |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --speculative-config '{"method": "draft_model", "draft_model": "meta-llama/Llama-3.1-8B-Instruct", "num_speculative_tokens": 5}'
```
> 使用 Llama-3.1-8B 作为 draft model 对 Llama-3.1-70B 进行投机解码，每次预测 5 个 token，可以显著降低生成延迟。

---

## `--structured-outputs-config`

| 属性 | 值 |
|------|-----|
| **说明** | 结构化输出引擎的配置。控制 vLLM 如何实现 JSON Schema 约束生成、正则表达式约束等结构化输出功能的底层引擎参数。以 JSON 字典形式传入 |
| **类型** | string (JSON dict) |
| **默认值** | 默认配置 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen3-0.6B --structured-outputs-config '{"backend": "xgrammar"}'
```
> 指定结构化输出使用 xgrammar 后端引擎，以支持 JSON Schema 等约束生成。

---

## `--kv-transfer-config`

| 属性 | 值 |
|------|-----|
| **说明** | 分布式 KV Cache 传输配置。在 Disaggregated Prefill/Decode 架构中，prefill 节点和 decode 节点之间需要传输 KV Cache 数据。此参数用于配置传输协议、地址、缓冲区大小等。以 JSON 字典形式传入 |
| **类型** | string (JSON dict) |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen3-0.6B --kv-transfer-config '{"kv_connector": "PyNcclConnector", "kv_role": "kv_producer", "kv_port": 14579}'
```
> 配置当前节点为 KV Cache 的生产者（prefill 节点），使用 PyNccl 进行高速 KV Cache 传输。

---

## `--kv-events-config`

| 属性 | 值 |
|------|-----|
| **说明** | KV 事件发布配置。当 KV Cache 发生创建、驱逐等事件时，可以将这些事件发布到外部系统（如消息队列），用于 KV Cache 的外部管理和调度。以 JSON 字典形式传入 |
| **类型** | string (JSON dict) |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen3-0.6B --kv-events-config '{"publisher": "zmq", "topic": "kv-events", "address": "tcp://10.0.0.1:5555"}'
```
> 将 KV Cache 事件通过 ZMQ 发布到指定地址，用于外部系统监控和管理 KV Cache 的生命周期。

---

## `--ec-transfer-config`

| 属性 | 值 |
|------|-----|
| **说明** | 分布式 EC（Erasure Coding）Cache 传输配置。与 `--kv-transfer-config` 类似，但使用纠删码技术来提高分布式缓存传输的容错性和效率。以 JSON 字典形式传入 |
| **类型** | string (JSON dict) |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen3-0.6B --ec-transfer-config '{"ec_connector": "PyNcclConnector", "ec_role": "kv_producer", "ec_port": 14580}'
```
> 配置 EC Cache 传输，使用纠删码机制在分布式节点之间传输缓存数据，提供更高的容错能力。

---
