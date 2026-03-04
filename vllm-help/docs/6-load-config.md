# 6 模型加载配置（LoadConfig）

> 返回 [README 总览](../README.md)

控制模型权重如何从存储加载到 GPU。包括加载格式选择、下载目录、文件过滤和加载策略等配置。

---

## `--load-format`

| 属性 | 值 |
|------|-----|
| **说明** | 权重加载格式。`auto`（推荐）自动选择 safetensors 或 pytorch 格式。其他选项包括 `gguf`（用于量化模型）、`bitsandbytes`（动态量化加载）、`tensorizer`（高速序列化加载）等 |
| **类型** | enum |
| **可选值** | `auto` / `safetensors` / `pytorch` / `gguf` / `bitsandbytes` / `tensorizer` |
| **默认值** | `auto` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --load-format safetensors
```
> 强制使用 safetensors 格式加载模型权重，跳过自动检测逻辑。适用于模型同时包含 safetensors 和 pytorch 文件时明确指定格式。

---

## `--download-dir`

| 属性 | 值 |
|------|-----|
| **说明** | 模型下载和加载目录。未设置时默认使用 HuggingFace 缓存目录（通常为 `~/.cache/huggingface`）。平台部署时通常指向共享存储路径 |
| **类型** | string |
| **默认值** | None（使用 HuggingFace 默认缓存目录） |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --download-dir /data/models
```
> 将模型下载到 `/data/models` 目录。适用于平台统一管理模型存储路径的场景，例如挂载 NFS 或高速 SSD。

---

## `--ignore-patterns`

| 属性 | 值 |
|------|-----|
| **说明** | 加载模型时忽略的文件 glob 模式列表。可以过滤掉不需要的文件（如原始未转换的权重文件），减少加载时间和磁盘占用 |
| **类型** | string |
| **默认值** | `['original/**/*']` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --ignore-patterns "*.bin" "original/**/*"
```
> 加载时忽略所有 `.bin` 文件和 `original/` 目录下的文件。适用于模型仓库中同时存在多种格式权重文件，只想加载 safetensors 的场景。

---

## `--model-loader-extra-config`

| 属性 | 值 |
|------|-----|
| **说明** | 模型加载器的额外配置，以 JSON 格式传入。不同的 `--load-format` 支持不同的额外配置项，例如 tensorizer 格式需要指定序列化路径等 |
| **类型** | string |
| **默认值** | `{}` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --load-format bitsandbytes --model-loader-extra-config '{"load_in_4bit": true}'
```
> 使用 bitsandbytes 格式加载模型时，通过额外配置启用 4-bit 量化。不同加载器支持的配置项不同。

---

## `--pt-load-map-location`

| 属性 | 值 |
|------|-----|
| **说明** | PyTorch checkpoint 的设备映射位置。控制 `torch.load()` 时将张量映射到哪个设备。通常使用 `cpu` 以避免在加载阶段占用 GPU 显存 |
| **类型** | string |
| **默认值** | `cpu` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve my-model --load-format pytorch --pt-load-map-location cpu
```
> 加载 PyTorch 格式的 checkpoint 时，先将权重映射到 CPU 内存，之后再转移到 GPU。这是默认行为，可以避免加载过程中 GPU OOM。

---

## `--safetensors-load-strategy`

| 属性 | 值 |
|------|-----|
| **说明** | Safetensors 文件的加载策略。`lazy` 使用内存映射按需加载，推荐本地高速存储；`eager` 一次性读入内存，推荐网络存储（NFS/对象存储）以减少随机 I/O；`torchao` 使用 TorchAO 加载策略 |
| **类型** | enum |
| **可选值** | `lazy` / `eager` / `torchao` |
| **默认值** | `lazy` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B --safetensors-load-strategy eager
```
> 使用 eager 策略一次性将 safetensors 文件读入内存。适用于模型文件存储在 NFS 或对象存储上的场景，避免内存映射带来的大量随机 I/O。

---

## `--use-tqdm-on-load`

| 属性 | 值 |
|------|-----|
| **说明** | 模型权重加载时是否显示 tqdm 进度条。启用后可以在终端看到加载进度，方便监控大模型的加载过程 |
| **类型** | bool |
| **默认值** | `True` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-70B --no-use-tqdm-on-load
```
> 禁用加载进度条。适用于生产环境中不需要终端输出进度信息的场景，或将日志重定向到文件时避免产生大量进度输出。
