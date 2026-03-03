# 8.3 编码器与性能

> 返回 [README 总览](../README.md)

多模态编码器的并行策略和性能优化配置，包括编码器的张量并行模式、attention 后端选择和视频 token 剪枝等。

---

## `--mm-encoder-tp-mode`

| 属性 | 值 |
|------|-----|
| **说明** | 多模态编码器（如 Vision Transformer）在多 GPU 张量并行场景下的并行模式。`weights` 模式将编码器的权重拆分到多张 GPU 上，每张 GPU 处理完整输入但只持有部分权重；`data` 模式则让每张 GPU 持有完整权重但只处理部分输入数据。对于较小的视觉编码器，`data` 模式通常更高效 |
| **类型** | enum |
| **可选值** | `weights` / `data` |
| **默认值** | `weights` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct -tp 4 --mm-encoder-tp-mode data
```
> 在 4 卡张量并行场景下，使用数据并行模式运行视觉编码器，每张 GPU 处理不同的图片输入。

---

## `--mm-encoder-attn-backend`

| 属性 | 值 |
|------|-----|
| **说明** | 多模态编码器使用的 attention 计算后端。可以指定特定的高性能 attention 实现（如 `FLASH_ATTN`），以优化视觉编码器的推理速度。留空则由引擎自动选择最优后端 |
| **类型** | string |
| **默认值** | None（自动选择） |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --mm-encoder-attn-backend FLASH_ATTN
```
> 强制多模态编码器使用 Flash Attention 后端，通常可以获得更好的推理性能。

---

## `--mm-processor-kwargs`

| 属性 | 值 |
|------|-----|
| **说明** | 传递给模型多模态 processor 的额外关键字参数，以 JSON 字典形式指定。这些参数直接传递给 HuggingFace 的 processor（如 `AutoProcessor`），可以用于控制图片预处理的具体行为，例如裁剪策略、分辨率设置等 |
| **类型** | string (JSON dict) |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --mm-processor-kwargs '{"min_pixels": 256, "max_pixels": 1280}'
```
> 向模型的多模态 processor 传递自定义的像素范围参数，控制图片预处理的分辨率。

---

## `--skip-mm-profiling`

| 属性 | 值 |
|------|-----|
| **说明** | 跳过多模态内存 profiling 阶段。vLLM 启动时会对多模态输入进行内存使用分析以合理分配资源，此过程可能较耗时。开启此选项可以加速服务启动，但用户需要自行确保显存足够处理多模态输入，否则可能在运行时出现 OOM |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --skip-mm-profiling
```
> 跳过启动时的多模态内存分析，缩短服务启动时间。适用于已充分了解模型显存需求的场景。

---

## `--video-pruning-rate`

| 属性 | 值 |
|------|-----|
| **说明** | 视频 token 剪枝率，取值范围 [0, 1)。通过丢弃一部分视频 token 来减少计算量和显存占用。例如设置为 0.5 表示丢弃约 50% 的视频 token。较高的剪枝率可以显著降低视频推理的资源消耗，但可能影响模型对视频内容的理解精度 |
| **类型** | float |
| **默认值** | None（不剪枝） |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --video-pruning-rate 0.3
```
> 将视频 token 剪枝率设为 0.3，丢弃约 30% 的视频 token，在保持合理理解精度的同时节省计算资源。

---
