# 5.7 其他并行选项

> 返回 [README 总览](../README.md)

并行配置中的其他选项，包括 Worker 类配置、通信优化和性能分析等。

---

## `--worker-cls`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义 Worker 类的全限定名。Worker 是 vLLM 中负责实际模型推理计算的组件，每张 GPU 对应一个 Worker 实例。通过自定义 Worker 类可以修改推理逻辑、添加自定义预处理/后处理等。设为 `auto` 时由 vLLM 根据运行环境自动选择合适的 Worker 类 |
| **类型** | string |
| **默认值** | `auto` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --worker-cls mypackage.custom_worker.MyWorker
```
> 使用自定义 Worker 类 `mypackage.custom_worker.MyWorker` 替代默认实现，用于特殊的推理逻辑定制。

---

## `--worker-extension-cls`

| 属性 | 值 |
|------|-----|
| **说明** | Worker 扩展类的全限定名。与 `--worker-cls` 不同，扩展类不替代默认 Worker，而是在其基础上添加额外功能（如自定义监控、日志、性能采集等）。这种插件式设计允许在不修改核心推理逻辑的情况下扩展 Worker 行为 |
| **类型** | string |
| **默认值** | 空 |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --worker-extension-cls mypackage.extensions.MetricsExtension
```
> 加载自定义 Worker 扩展类，在推理过程中采集额外的性能指标。

---

## `--disable-custom-all-reduce`

| 属性 | 值 |
|------|-----|
| **说明** | 禁用 vLLM 自定义的 all-reduce 通信实现，回退到 NCCL 原生的 all-reduce。vLLM 默认使用优化过的自定义 all-reduce 实现以提升多 GPU 通信效率。在某些硬件配置或网络拓扑下，自定义实现可能不稳定，此时可回退到经过广泛验证的 NCCL 实现 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --disable-custom-all-reduce
```
> 禁用自定义 all-reduce，使用 NCCL 原生实现。适用于自定义 all-reduce 在当前硬件上出现兼容性问题的情况。

---

## `--ray-workers-use-nsight`

| 属性 | 值 |
|------|-----|
| **说明** | 使 Ray worker 进程在 NVIDIA Nsight Systems 性能分析器下运行。Nsight Systems 是 NVIDIA 提供的系统级性能分析工具，可以详细记录 GPU 计算、内存传输、CUDA 内核执行等信息。启用后会产生性能分析数据文件，用于深度性能调优 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B-Instruct --tensor-parallel-size 4 --distributed-executor-backend ray --ray-workers-use-nsight
```
> 在 Ray 分布式推理模式下启用 Nsight 性能分析，采集每个 GPU worker 的详细性能数据，用于排查性能瓶颈。

---

## `--enable-multimodal-encoder-data-parallel`

| 属性 | 值 |
|------|-----|
| **说明** | 对多模态模型的编码器启用数据并行。在多模态模型（如视觉语言模型）中，编码器（如 ViT 视觉编码器）和语言模型可以采用不同的并行策略。启用此选项后，编码器使用数据并行（多个 GPU 各处理不同的图片），而语言模型部分仍使用张量并行，从而优化多模态推理效率 |
| **类型** | bool |
| **默认值** | — |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve llava-hf/llava-v1.6-mistral-7b-hf --tensor-parallel-size 4 --enable-multimodal-encoder-data-parallel
```
> 部署多模态模型 LLaVA 时，视觉编码器使用数据并行处理不同图片，语言模型使用张量并行，优化整体推理效率。

---
