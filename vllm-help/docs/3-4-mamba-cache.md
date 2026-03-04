# 3.4 Mamba 模型缓存

> 返回 [README 总览](../README.md)

Mamba 是一种基于状态空间模型（SSM）的替代 Transformer 架构。这些参数专门用于配置 Mamba 模型的缓存行为。

---

## `--mamba-block-size`

| 属性 | 值 |
|------|-----|
| **说明** | Mamba 模型缓存块大小，以 token 数为单位。类似于 Transformer 架构中的 `--block-size`，但专门针对 Mamba 模型。该值必须为 8 的倍数，以满足 Mamba 架构的对齐要求 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve state-spaces/mamba-2.8b --mamba-block-size 64
```
> 为 Mamba 模型设置 64 token 的缓存块大小，平衡内存碎片与管理开销。

---

## `--mamba-cache-dtype`

| 属性 | 值 |
|------|-----|
| **说明** | Mamba 模型缓存（卷积状态缓存）的数据类型。`auto` 表示自动匹配模型精度。可以手动指定以在精度和显存占用之间做出权衡 |
| **类型** | string |
| **默认值** | auto |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve state-spaces/mamba-2.8b --mamba-cache-dtype auto
```
> 使用自动检测的数据类型存储 Mamba 卷积状态缓存，与模型权重精度保持一致。

---

## `--mamba-ssm-cache-dtype`

| 属性 | 值 |
|------|-----|
| **说明** | Mamba SSM（Selective State Space Model）状态缓存的数据类型。SSM 状态缓存存储了序列的隐藏状态信息，`auto` 表示自动匹配模型精度 |
| **类型** | string |
| **默认值** | auto |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve state-spaces/mamba-2.8b --mamba-ssm-cache-dtype auto
```
> 使用自动检测的数据类型存储 Mamba SSM 状态缓存，确保与模型的计算精度一致以避免精度损失。

---
