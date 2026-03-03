# 7 LoRA 配置（LoRAConfig）

> 返回 [README 总览](../README.md)

LoRA（Low-Rank Adaptation）是当前最流行的模型微调方法。vLLM 支持在推理时动态加载 LoRA 适配器，让一个基础模型同时服务多个微调版本。

---

## `--enable-lora`

| 属性 | 值 |
|------|-----|
| **说明** | 启用 LoRA 支持。开启后可以在推理时动态加载和切换 LoRA 适配器，让同一个基础模型同时服务多个微调版本的请求 |
| **类型** | bool |
| **默认值** | None（未启用） |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --enable-lora --lora-modules my-adapter=./lora-weights
```
> 启用 LoRA 支持并预加载一个名为 `my-adapter` 的 LoRA 适配器。用户可以在 API 请求中通过 model 参数指定使用 `my-adapter` 进行推理。

---

## `--max-loras`

| 属性 | 值 |
|------|-----|
| **说明** | 单个 batch 中最大 LoRA 数量，即同时可以使用多少个不同的 LoRA 适配器。当多个用户同时使用不同的微调版本时，需要增大此值 |
| **类型** | int |
| **默认值** | 1 |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --enable-lora --max-loras 4
```
> 允许同一个 batch 中同时使用最多 4 个不同的 LoRA 适配器。适用于多租户场景，每个租户使用各自微调的模型版本。

---

## `--max-lora-rank`

| 属性 | 值 |
|------|-----|
| **说明** | 最大 LoRA 秩。LoRA 秩决定了适配器的参数量和表达能力——秩越高，微调效果越好但显存开销越大。此参数需要大于等于所加载 LoRA 适配器的实际秩 |
| **类型** | enum |
| **可选值** | `1` / `8` / `16` / `32` / `64` / `128` / `256` / `320` / `512` |
| **默认值** | 16 |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --enable-lora --max-lora-rank 64
```
> 设置最大 LoRA 秩为 64。如果你的 LoRA 适配器使用 rank=64 训练，则此值必须至少设为 64，否则加载会失败。

---

## `--max-cpu-loras`

| 属性 | 值 |
|------|-----|
| **说明** | CPU 内存中最大 LoRA 数量。用于在 CPU 内存中缓存不活跃的 LoRA 适配器，减少从磁盘加载的开销。此值需要大于等于 `--max-loras` |
| **类型** | int |
| **默认值** | None（与 `--max-loras` 相同） |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --enable-lora --max-loras 2 --max-cpu-loras 8
```
> GPU 上同时保持 2 个 LoRA，CPU 内存中缓存最多 8 个 LoRA。当请求需要的 LoRA 不在 GPU 上但在 CPU 缓存中时，可以快速加载到 GPU，而不需要从磁盘重新读取。

---

## `--lora-dtype`

| 属性 | 值 |
|------|-----|
| **说明** | LoRA 适配器的数据类型。`auto` 会自动匹配基础模型的数据类型。显式指定可以控制 LoRA 计算的精度和显存占用 |
| **类型** | enum |
| **可选值** | `auto` / `float16` / `bfloat16` |
| **默认值** | `auto` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --enable-lora --lora-dtype bfloat16
```
> 强制使用 bfloat16 精度加载 LoRA 权重。适用于基础模型为 bfloat16 而 LoRA 文件存储为 float32 时，强制降精度以节省显存。

---

## `--fully-sharded-loras`

| 属性 | 值 |
|------|-----|
| **说明** | 启用完全分片 LoRA 计算。在多 GPU 并行场景下，将 LoRA 计算也进行分片，避免单卡瓶颈。在高并发或高 LoRA rank 场景下可能获得更好的性能 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B --enable-lora --tensor-parallel-size 4 --fully-sharded-loras
```
> 在 4 卡张量并行场景下启用完全分片 LoRA。LoRA 计算将均匀分布到所有 GPU 上，而不是在单卡上集中计算，适合高 rank 或高并发 LoRA 请求。

---

## `--lora-extra-vocab-size` [已废弃]

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] LoRA 适配器额外词表的最大大小。当 LoRA 微调时扩展了词表（如添加了特殊 token），需要此参数预留空间。此参数已废弃，将在未来版本中移除 |
| **类型** | int |
| **默认值** | 256 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --enable-lora --lora-extra-vocab-size 512
```
> 为 LoRA 适配器预留 512 个额外词表空间。如果微调时添加了超过 256 个新 token，需要增大此值。注意此参数已废弃，不建议在新项目中使用。

---

## `--default-mm-loras`

| 属性 | 值 |
|------|-----|
| **说明** | 多模态默认 LoRA 映射配置。用于为多模态模型的不同组件指定默认的 LoRA 适配器 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve llava-hf/llava-v1.6-7b --enable-lora --default-mm-loras '{"vision_encoder": "my-vision-lora"}'
```
> 为多模态模型的视觉编码器组件指定一个默认的 LoRA 适配器。当请求未明确指定 LoRA 时，视觉编码器将自动使用此适配器。
