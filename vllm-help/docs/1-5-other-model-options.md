# 1.5 其他模型选项

> 返回 [README 总览](../README.md)

模型配置中的其他高级选项，大多面向平台管理员。包括安全相关的开关、性能调优选项以及一些实验性功能。

---

## `--code-revision`

| 属性 | 值 |
|------|-----|
| **说明** | 模型代码的版本（branch/tag/commit id）。与 `--revision` 类似，但专门用于锁定模型自定义代码（如自定义 modeling 文件）的版本，与模型权重版本独立管理 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model THUDM/chatglm3-6b --trust-remote-code --code-revision v1.0
```
> 锁定模型自定义代码到 `v1.0` 版本，防止远程代码更新引入不兼容变更。

---

## `--hf-config-path`

| 属性 | 值 |
|------|-----|
| **说明** | 自定义 HuggingFace config 路径。当模型的配置文件不在默认位置时，可通过此参数指定 `config.json` 的路径 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model /data/models/my-model --hf-config-path /data/configs/custom-config
```
> 从自定义路径加载模型的 HuggingFace 配置文件。

---

## `--hf-overrides`

| 属性 | 值 |
|------|-----|
| **说明** | 覆盖 HuggingFace 配置参数。以 JSON 格式传入，可以修改模型 `config.json` 中的任意字段，无需修改原始文件 |
| **类型** | string |
| **默认值** | `{}` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --hf-overrides '{"hidden_size": 4096}'
```
> 覆盖模型配置中的 `hidden_size` 参数为 4096。

---

## `--skip-tokenizer-init`

| 属性 | 值 |
|------|-----|
| **说明** | 跳过 tokenizer 初始化。启用后，引擎不会加载 tokenizer，仅接受 token id 作为输入。适用于已在客户端完成分词的场景 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --skip-tokenizer-init
```
> 跳过 tokenizer 加载，引擎将直接接受 token id 输入而非文本。

---

## `--override-pooler-config`

| 属性 | 值 |
|------|-----|
| **说明** | [已废弃] Pooler 配置覆盖。以 JSON 格式传入，用于覆盖 pooling 模型的默认配置。建议使用 `--pooler-config` 替代 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model BAAI/bge-large-en-v1.5 --override-pooler-config '{"pooling_type": "mean"}'
```
> 覆盖 pooler 配置，使用 mean pooling 策略。此参数已废弃，建议使用 `--pooler-config`。

---

## `--pooler-config`

| 属性 | 值 |
|------|-----|
| **说明** | Pooler 配置（用于 pooling 模型）。以 JSON 格式传入，定义 pooling 层的行为方式，如 pooling 策略、归一化等 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model BAAI/bge-large-en-v1.5 --pooler-config '{"pooling_type": "cls"}'
```
> 配置 pooling 模型使用 CLS token 的输出作为句子嵌入。

---

## `--override-attention-dtype`

| 属性 | 值 |
|------|-----|
| **说明** | 覆盖 attention 层的数据类型。允许单独控制 attention 计算的精度，与模型整体精度（`--dtype`）独立设置。可用于在特定场景下优化性能或精度 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --override-attention-dtype float32
```
> 将 attention 计算的精度设置为 float32，提升 attention 层的数值稳定性。

---

## `--enable-prompt-embeds`

| 属性 | 值 |
|------|-----|
| **说明** | 允许通过 API 传入文本 embeddings（而非文本）。存在安全风险——恶意构造的 embeddings 可能导致引擎崩溃或产生不可预期的输出。仅限受信任的用户使用 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --enable-prompt-embeds
```
> 启用 prompt embeddings 输入支持。仅在受信环境中使用，存在安全风险。

---

## `--enable-sleep-mode`

| 属性 | 值 |
|------|-----|
| **说明** | 启用引擎休眠模式（仅 CUDA/HIP 平台）。当引擎空闲时释放 GPU 显存，收到新请求时重新加载。适用于多模型共享 GPU 资源的场景 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --enable-sleep-mode
```
> 启用休眠模式，引擎空闲时自动释放 GPU 显存以供其他服务使用。

---

## `--disable-cascade-attn`

| 属性 | 值 |
|------|-----|
| **说明** | 禁用 cascade attention 优化。Cascade attention 是一种用于加速长序列推理的优化技术，禁用后会回退到标准 attention 实现 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model meta-llama/Llama-3.1-70B-Instruct --disable-cascade-attn
```
> 禁用 cascade attention 优化，使用标准 attention 实现。

---

## `--disable-sliding-window`

| 属性 | 值 |
|------|-----|
| **说明** | 禁用滑动窗口注意力机制。某些模型（如 Mistral）使用滑动窗口来限制 attention 的范围以节省计算。禁用后每个 token 可以关注所有历史 token |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model mistralai/Mistral-7B-Instruct-v0.3 --disable-sliding-window
```
> 禁用 Mistral 模型的滑动窗口 attention，使其使用全局 attention。

---

## `--io-processor-plugin`

| 属性 | 值 |
|------|-----|
| **说明** | IO 处理器插件。指定自定义的输入/输出处理器，用于在请求进入引擎前和输出返回前进行预处理和后处理 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen3-0.6B --io-processor-plugin my_plugin.CustomIOProcessor
```
> 加载自定义 IO 处理器插件，对输入输出进行额外处理。

---

## `--allowed-local-media-path`

| 属性 | 值 |
|------|-----|
| **说明** | 允许 API 读取的本地媒体路径。多模态模型可以从本地文件系统加载图片/视频。此参数限制可读取的目录范围，防止未授权的文件访问。存在安全风险，应谨慎设置 |
| **类型** | string |
| **默认值** | 空 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen2.5-VL-7B-Instruct --allowed-local-media-path /data/media
```
> 允许多模态模型从 `/data/media` 目录加载本地图片和视频文件。

---

## `--allowed-media-domains`

| 属性 | 值 |
|------|-----|
| **说明** | 允许的外部媒体域名白名单。限制多模态模型可以从哪些外部域名获取图片/视频等媒体资源，防止 SSRF 等安全问题 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --model Qwen/Qwen2.5-VL-7B-Instruct --allowed-media-domains "cdn.example.com,images.example.org"
```
> 仅允许从 `cdn.example.com` 和 `images.example.org` 获取外部媒体资源。

---
