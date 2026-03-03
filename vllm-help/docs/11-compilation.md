# 11 编译优化配置（CompilationConfig）

> 返回 [README 总览](../README.md)

通过 `torch.compile` 和 CUDA Graph 优化推理性能。这些是深度性能调优选项，可以显著提升推理速度。

---

## `--compilation-config` / `-O`

| 属性 | 值 |
|------|-----|
| **说明** | 完整的编译配置，以 JSON 格式传入。支持快捷方式语法 `-O.mode=3` 直接设置编译模式级别。不同的 mode 级别对应不同的编译优化强度：级别越高优化越激进，启动时间越长但推理越快 |
| **类型** | string |
| **默认值** | 默认配置 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B -O.mode=3
```
> 使用级别 3 的编译优化模式启动服务。这会启用 `torch.compile` 对模型进行深度编译优化，首次启动会较慢（需要编译），但后续推理速度显著提升。

---

## `--cudagraph-capture-sizes`

| 属性 | 值 |
|------|-----|
| **说明** | CUDA Graph 捕获的 batch 大小列表。CUDA Graph 会预先录制特定 batch 大小的 GPU 计算图，推理时直接回放以减少 CPU 开销。只有列表中的 batch 大小会使用 CUDA Graph 加速，其他大小回退到普通执行 |
| **类型** | string |
| **默认值** | 自动推断 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --cudagraph-capture-sizes 1,2,4,8,16,32
```
> 预先捕获 batch 大小为 1、2、4、8、16、32 的 CUDA Graph。当实际推理 batch 匹配这些大小时，将使用预录制的计算图，减少 kernel launch 开销，提升吞吐量。

---

## `--max-cudagraph-capture-size`

| 属性 | 值 |
|------|-----|
| **说明** | CUDA Graph 最大捕获大小。限制 CUDA Graph 可以捕获的最大 batch 大小。超过此大小的 batch 将不使用 CUDA Graph 加速，回退到普通的 eager 执行。增大此值会消耗更多 GPU 显存用于存储捕获的计算图 |
| **类型** | int |
| **默认值** | 自动推断 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --max-cudagraph-capture-size 64
```
> 将 CUDA Graph 最大捕获大小设为 64。batch 大小在 64 以内的推理将尝试使用 CUDA Graph 加速，超过 64 的大 batch 则回退到普通执行模式。

---

## `--cuda-graph-sizes` [已废弃]

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] CUDA Graph 捕获大小列表。此参数已被 `--cudagraph-capture-sizes` 替代，将在未来版本中移除。功能与 `--cudagraph-capture-sizes` 相同 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --cuda-graph-sizes 1,2,4,8,16
```
> 设置 CUDA Graph 捕获大小。注意此参数已废弃，请使用 `--cudagraph-capture-sizes` 替代。
