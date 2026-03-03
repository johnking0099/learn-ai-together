# vLLM Serve 配置参数完全指南

> 基于 vLLM v0.11.2 的 `vllm serve --help` 输出整理。
> 面向产品经理与平台工程师，帮助理解每个配置选项的用途，并为构建模型部署服务提供分层暴露建议。

---

## 阅读说明

### 什么是 vLLM？

vLLM 是当前最流行的开源大语言模型（LLM）推理引擎之一。`vllm serve` 命令会启动一个 **兼容 OpenAI API 的 HTTP 服务**，让用户可以像调用 OpenAI API 一样调用本地或私有部署的大模型。

### 参数暴露分层建议

当基于 vLLM 构建一个面向用户的模型部署平台时，不同参数应该有不同的暴露策略：

| 标记 | 含义 | 说明 |
|------|------|------|
| **普通用户** | 应在 UI 上暴露给所有用户 | 用户必须或经常需要设置的选项，如选择哪个模型、设置上下文长度等 |
| **高级用户** | 仅对高级用户/开发者暴露 | 需要一定技术理解才能正确配置的选项，如量化方式、LoRA 配置等 |
| **平台管理** | 由平台/运维团队预设，不暴露给用户 | 涉及基础设施、安全、网络等底层配置，用户无需也不应修改 |

---

## 目录

1. [模型配置（ModelConfig）](#1-模型配置modelconfig) — 选择和配置模型
2. [前端服务配置（Frontend）](#2-前端服务配置frontend) — API 服务器行为
3. [KV Cache 配置（CacheConfig）](#3-kv-cache-配置cacheconfig) — 显存与缓存管理
4. [调度器配置（SchedulerConfig）](#4-调度器配置schedulerconfig) — 请求调度策略
5. [并行配置（ParallelConfig）](#5-并行配置parallelconfig) — 分布式与并行推理
6. [模型加载配置（LoadConfig）](#6-模型加载配置loadconfig) — 权重加载方式
7. [LoRA 配置（LoRAConfig）](#7-lora-配置loraconfig) — 微调适配器
8. [多模态配置（MultiModalConfig）](#8-多模态配置multimodalconfig) — 图片/视频等多模态输入
9. [结构化输出配置（StructuredOutputsConfig）](#9-结构化输出配置structuredoutputsconfig) — JSON Schema 等约束输出
10. [可观测性配置（ObservabilityConfig）](#10-可观测性配置observabilityconfig) — 监控与追踪
11. [编译优化配置（CompilationConfig）](#11-编译优化配置compilationconfig) — 性能编译优化
12. [全局配置（VllmConfig 与通用选项）](#12-全局配置vllmconfig-与通用选项) — 顶层与杂项配置

---

## 1. 模型配置（ModelConfig）

> 这是最核心的配置组。决定了"部署什么模型"以及"模型如何运行"。

### 1.1 [模型选择与标识](docs/1-1-model-selection.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `model_tag` (位置参数) | 要部署的模型标签（如 `Qwen/Qwen3-0.6B`），也可通过 config 文件指定 | None | **普通用户** |
| `--model` | 模型的 HuggingFace 名称或本地路径。这是最核心的参数——用户通过它指定想部署哪个模型 | `Qwen/Qwen3-0.6B` | **普通用户** |
| `--served-model-name` | API 中暴露的模型名称。用户调用 API 时用此名称引用模型。支持设置多个别名 | 与 `--model` 相同 | **普通用户** |
| `--revision` | 模型的具体版本（branch/tag/commit id）。用于锁定特定版本避免意外更新 | None（使用默认版本） | **高级用户** |
| `--tokenizer` | 自定义 tokenizer 的名称或路径。大多数情况下无需设置，使用模型自带的即可 | 与 model 相同 | **高级用户** |
| `--tokenizer-revision` | Tokenizer 的版本号 | None | **平台管理** |
| `--tokenizer-mode` | Tokenizer 模式：`auto`/`slow`/`mistral`/`custom` | `auto` | **平台管理** |
| `--hf-token` | HuggingFace 访问令牌，用于下载需要授权的模型（如 Llama 系列） | None | **平台管理** |

**产品设计建议：**
- `--model` 是用户必须选择的核心参数，建议做成模型选择器（下拉列表或搜索框），展示模型名称、大小、能力描述等。
- `--served-model-name` 允许用户给自己的部署起一个自定义名称，方便业务区分。
- `--hf-token` 涉及凭证，应由平台统一管理（如通过 Secret 注入），不应让用户在 UI 中输入。

### 1.2 [模型运行参数](docs/1-2-model-runtime.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--max-model-len` | **模型上下文长度**（prompt + output 的总 token 数）。这是用户最关心的参数之一：它决定了模型能处理多长的输入+输出。支持 `1k`/`2M` 等可读格式 | 自动从模型配置推导 | **普通用户** |
| `--dtype` | 模型权重和激活值的数据类型。`auto`（推荐）/`float16`/`bfloat16`/`float32`。影响显存占用和推理精度 | `auto` | **高级用户** |
| `--quantization` / `-q` | 模型量化方式。量化可以大幅减少显存占用，但会轻微降低精度。常见方式有 AWQ、GPTQ 等 | None（不量化） | **高级用户** |
| `--seed` | 随机种子，用于保证结果可复现。V1 版本默认为 0 | None | **高级用户** |
| `--enforce-eager` | 是否强制使用 eager 模式（禁用 CUDA Graph）。调试时有用，但会降低性能 | `False` | **平台管理** |
| `--trust-remote-code` | 是否信任远程代码。某些模型需要执行 HuggingFace 上的自定义代码才能加载 | `False` | **平台管理** |
| `--model-impl` | 模型实现方式：`auto`/`vllm`/`transformers`/`terratorch` | `auto` | **平台管理** |
| `--runner` | 模型 runner 类型：`auto`/`generate`/`pooling`/`draft` | `auto` | **平台管理** |
| `--config-format` | 模型配置格式：`auto`/`hf`/`mistral` | `auto` | **平台管理** |

**产品设计建议：**
- `--max-model-len` 对用户非常重要。建议在 UI 上清晰展示，并标注该模型支持的最大值以及当前设置对显存的影响。例如："上下文长度：8192 tokens（最大支持 128K，更长上下文需要更多显存）"。
- `--quantization` 和 `--dtype` 对高级用户有意义。可以简化为"精度模式"选项：全精度 / 半精度 / 量化（显存省 50%+），让用户在"质量"与"资源"之间做权衡。
- `--trust-remote-code` 有安全隐患，平台应根据模型来源策略统一决定，不应暴露给用户。

### 1.3 [生成配置](docs/1-3-generation-config.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--generation-config` | 生成配置来源。`auto` 从模型加载，`vllm` 使用 vLLM 默认值，或指定文件夹路径 | `auto` | **高级用户** |
| `--override-generation-config` | 覆盖生成配置，如 `{"temperature": 0.5}`。可设置全局默认的生成参数 | `{}` | **高级用户** |
| `--max-logprobs` | 返回的最大 log probabilities 数量。-1 表示不限制（可能导致 OOM） | 20 | **平台管理** |
| `--logprobs-mode` | Logprobs 内容类型：`raw_logprobs`（原始）/`processed_logprobs`（处理后）等 | `raw_logprobs` | **平台管理** |
| `--logits-processor-pattern` | 允许的 logits processor 正则匹配模式 | None（不允许） | **平台管理** |
| `--logits-processors` | 预加载的 logits processors | None | **平台管理** |

### 1.4 [任务与转换](docs/1-4-task-conversion.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--task` | [已废弃] 模型任务类型：`generate`/`embed`/`classify`/`reward`/`score` 等 | None | **高级用户** |
| `--convert` | 模型适配器转换：如将文本生成模型适配为 pooling 任务 | `auto` | **平台管理** |

### 1.5 [其他模型选项](docs/1-5-other-model-options.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--code-revision` | 模型代码的版本 | None | **平台管理** |
| `--hf-config-path` | 自定义 HF config 路径 | None | **平台管理** |
| `--hf-overrides` | 覆盖 HuggingFace 配置参数 | `{}` | **平台管理** |
| `--skip-tokenizer-init` | 跳过 tokenizer 初始化（仅接受 token id 输入） | `False` | **平台管理** |
| `--override-pooler-config` | [已废弃] Pooler 配置覆盖 | None | **平台管理** |
| `--pooler-config` | Pooler 配置（用于 pooling 模型） | None | **平台管理** |
| `--override-attention-dtype` | 覆盖 attention 的数据类型 | None | **平台管理** |
| `--enable-prompt-embeds` | 允许传入文本 embeddings（安全风险，仅限受信用户） | `False` | **平台管理** |
| `--enable-sleep-mode` | 启用引擎休眠模式（仅 CUDA/HIP 平台） | `False` | **平台管理** |
| `--disable-cascade-attn` | 禁用 cascade attention | `False` | **平台管理** |
| `--disable-sliding-window` | 禁用滑动窗口 | `False` | **平台管理** |
| `--io-processor-plugin` | IO 处理器插件 | None | **平台管理** |
| `--allowed-local-media-path` | 允许 API 读取的本地媒体路径（安全风险） | 空 | **平台管理** |
| `--allowed-media-domains` | 允许的外部媒体域名白名单 | None | **平台管理** |

---

## 2. 前端服务配置（Frontend）

> 控制 OpenAI 兼容 API 服务器的行为，包括网络、安全、日志、工具调用等。

### 2.1 [网络与访问](docs/2-1-network-access.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--host` | 监听的主机地址 | None | **平台管理** |
| `--port` | 监听的端口号 | 8000 | **平台管理** |
| `--uds` | Unix Domain Socket 路径。设置后忽略 host 和 port | None | **平台管理** |
| `--root-path` | FastAPI 的 root_path，用于反向代理场景 | None | **平台管理** |
| `--api-key` | API 密钥。设置后所有请求必须携带此密钥 | None | **平台管理** |

**产品设计建议：**
- 网络配置（host/port）应由平台统一管理，用户无需关心。
- `--api-key` 应由平台的认证系统统一管理，而非用户手动设置。

### 2.2 [CORS（跨域资源共享）](docs/2-2-cors.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--allowed-origins` | 允许的跨域来源 | `['*']` | **平台管理** |
| `--allowed-methods` | 允许的 HTTP 方法 | `['*']` | **平台管理** |
| `--allowed-headers` | 允许的请求头 | `['*']` | **平台管理** |
| `--allow-credentials` | 是否允许携带凭证 | `False` | **平台管理** |

### 2.3 [Chat Template 与工具调用](docs/2-3-chat-template-tools.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--chat-template` | 自定义 chat template 文件路径或模板字符串。控制模型如何理解多轮对话格式 | None（使用模型自带） | **高级用户** |
| `--chat-template-content-format` | 消息内容渲染格式：`auto`/`string`/`openai` | `auto` | **平台管理** |
| `--enable-auto-tool-choice` | 启用模型自动选择工具（Function Calling）。需配合 `--tool-call-parser` 使用 | `False` | **高级用户** |
| `--tool-call-parser` | 工具调用解析器。不同模型使用不同解析器（如 `hermes`/`llama3_json`/`mistral`/`qwen3_coder` 等） | None | **高级用户** |
| `--tool-parser-plugin` | 自定义工具解析器插件路径 | 空 | **平台管理** |
| `--tool-server` | 外部工具服务器地址（host:port 列表） | None | **高级用户** |
| `--exclude-tools-when-tool-choice-none` | 当 `tool_choice='none'` 时，从 prompt 中排除工具定义 | `False` | **平台管理** |
| `--trust-request-chat-template` | 是否信任请求中的 chat template（安全风险） | `False` | **平台管理** |

**产品设计建议：**
- Function Calling（工具调用）是当前 AI 应用的热门特性。建议在 UI 上提供开关："启用 Function Calling"，并根据所选模型自动匹配合适的 `tool-call-parser`。
- `--chat-template` 对于使用自定义对话格式的高级用户很有用，可以在"高级设置"中提供上传入口。

### 2.4 [SSL/TLS 安全](docs/2-4-ssl-tls.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--ssl-certfile` | SSL 证书文件路径 | None | **平台管理** |
| `--ssl-keyfile` | SSL 私钥文件路径 | None | **平台管理** |
| `--ssl-ca-certs` | CA 证书文件 | None | **平台管理** |
| `--ssl-cert-reqs` | 是否要求客户端证书 | 0 | **平台管理** |
| `--enable-ssl-refresh` | SSL 证书文件变更时自动刷新 | `False` | **平台管理** |

### 2.5 [日志与调试](docs/2-5-logging.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--uvicorn-log-level` | Uvicorn 日志级别：`debug`/`info`/`warning`/`error`/`critical`/`trace` | `info` | **平台管理** |
| `--disable-uvicorn-access-log` | 禁用 Uvicorn 访问日志 | `False` | **平台管理** |
| `--log-config-file` | 日志配置 JSON 文件路径 | None | **平台管理** |
| `--log-error-stack` | 是否记录错误的完整堆栈 | `False` | **平台管理** |
| `--max-log-len` | 日志中打印的最大 prompt 字符数 | None（不限制） | **平台管理** |
| `--enable-log-requests` | 启用请求日志记录 | `False` | **平台管理** |
| `--enable-log-outputs` | 记录模型输出（需先启用 `--enable-log-requests`） | `False` | **平台管理** |

### 2.6 [LoRA 模块加载（前端层面）](docs/2-6-lora-modules-frontend.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--lora-modules` | 预加载的 LoRA 模块。格式：`name=path` 或 JSON | None | **高级用户** |

### 2.7 [其他前端选项](docs/2-7-other-frontend.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--response-role` | `add_generation_prompt=true` 时返回的角色名 | `assistant` | **平台管理** |
| `--disable-frontend-multiprocessing` | 在同一进程中运行前端和引擎（调试用） | `False` | **平台管理** |
| `--disable-fastapi-docs` | 禁用 FastAPI 自带的 Swagger UI 文档 | `False` | **平台管理** |
| `--enable-force-include-usage` | 每次响应都包含 usage 信息 | `False` | **平台管理** |
| `--enable-prompt-tokens-details` | 在 usage 中包含 prompt_tokens_details | `False` | **平台管理** |
| `--enable-request-id-headers` | 响应中添加 X-Request-Id header | `False` | **平台管理** |
| `--enable-server-load-tracking` | 跟踪服务器负载指标 | `False` | **平台管理** |
| `--enable-tokenizer-info-endpoint` | 启用 `/get_tokenizer_info` 端点 | `False` | **平台管理** |
| `--return-tokens-as-token-ids` | 将 token 以 `token_id:{id}` 格式返回 | `False` | **平台管理** |
| `--middleware` | 自定义 ASGI 中间件 | `[]` | **平台管理** |
| `--tokens-only` | 仅启用 Tokens In/Out 端点（用于 Disaggregated 部署） | `False` | **平台管理** |
| `--h11-max-header-count` | HTTP 请求允许的最大 header 数量 | 256 | **平台管理** |
| `--h11-max-incomplete-event-size` | 不完整 HTTP 事件的最大大小 | 4194304 (4MB) | **平台管理** |

---

## 3. KV Cache 配置（CacheConfig）

> KV Cache 是 LLM 推理中最关键的内存组件。它缓存已经计算过的 attention 键值对，避免重复计算，直接影响吞吐量和能同时服务的请求数。

### 3.1 [核心内存参数](docs/3-1-core-memory.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--gpu-memory-utilization` | **GPU 显存利用率**。0~1 之间的比例，如 0.9 表示使用 90% 的显存。这是调节"能同时服务多少请求"的关键旋钮 | 0.9 | **高级用户** |
| `--kv-cache-dtype` | KV Cache 数据类型：`auto`/`fp8`/`bfloat16` 等。使用 fp8 可以显著节省显存但略有精度损失 | `auto` | **高级用户** |
| `--enable-prefix-caching` | **启用前缀缓存**。多个请求共享相同的 system prompt 时可以大幅提升性能。V1 默认开启 | None（V1 默认开启） | **高级用户** |

**产品设计建议：**
- `--gpu-memory-utilization` 可以简化为"显存使用策略"：保守(0.7) / 标准(0.9) / 激进(0.95)。
- `--enable-prefix-caching` 适合有大量相同 system prompt 的场景（如客服、助手类应用），建议在 UI 上说明其收益。

### 3.2 [高级缓存参数](docs/3-2-advanced-cache.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--kv-cache-memory-bytes` | 手动指定每 GPU 的 KV Cache 大小（字节）。比 gpu-memory-utilization 更精细 | None | **平台管理** |
| `--block-size` | 缓存块大小（token 数）：`1`/`8`/`16`/`32`/`64`/`128`/`256`。CUDA 上最大支持 32 | 自动推断 | **平台管理** |
| `--swap-space` | 每 GPU 的 CPU 交换空间大小（GiB）。当 GPU 显存不足时，将部分 KV Cache 交换到 CPU | 4 | **平台管理** |
| `--cpu-offload-gb` | 每 GPU 卸载到 CPU 的空间（GiB）。相当于虚拟扩展 GPU 显存，但需要快速 CPU-GPU 互联 | 0 | **平台管理** |
| `--num-gpu-blocks-override` | 手动覆盖 GPU blocks 数量（测试用） | None | **平台管理** |
| `--calculate-kv-scales` | fp8 KV Cache 时动态计算缩放因子 | `False` | **平台管理** |
| `--prefix-caching-hash-algo` | 前缀缓存的哈希算法：`sha256`/`sha256_cbor` | `sha256` | **平台管理** |

### 3.3 [KV Offloading](docs/3-3-kv-offloading.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--kv-offloading-backend` | KV Cache 卸载后端：`native`（vLLM 原生 CPU 卸载）/`lmcache` | None | **平台管理** |
| `--kv-offloading-size` | KV Cache 卸载缓冲区大小（GiB） | None | **平台管理** |

### 3.4 [Mamba 模型缓存](docs/3-4-mamba-cache.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--mamba-block-size` | Mamba 缓存块大小（需为 8 的倍数） | None | **平台管理** |
| `--mamba-cache-dtype` | Mamba 缓存数据类型 | `auto` | **平台管理** |
| `--mamba-ssm-cache-dtype` | Mamba SSM 状态缓存数据类型 | `auto` | **平台管理** |

### 3.5 [KV Sharing](docs/3-5-kv-sharing.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--kv-sharing-fast-prefill` | 实验性功能：KV sharing 场景下的快速 prefill（如 YOCO 架构） | `False` | **平台管理** |

---

## 4. 调度器配置（SchedulerConfig）

> 调度器决定了如何安排请求的处理顺序、如何分配 batch、如何平衡延迟与吞吐量。

### 4.1 [核心调度参数](docs/4-1-core-scheduling.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--max-num-seqs` | **单次迭代最大序列数**。即同时处理的请求数量上限。直接影响吞吐量和延迟 | 自动推断 | **高级用户** |
| `--max-num-batched-tokens` | **单次迭代最大 token 数**。控制每次前向传播处理的总 token 数。支持 `1k`/`2M` 格式 | 自动推断 | **高级用户** |
| `--scheduling-policy` | 调度策略：`fcfs`（先来先服务）/ `priority`（优先级调度） | `fcfs` | **高级用户** |

**产品设计建议：**
- `--max-num-seqs` 和 `--max-num-batched-tokens` 共同决定了"并发处理能力"。可以简化为一个"并发模式"选项：低延迟（少并发，快响应）/ 高吞吐（多并发，充分利用 GPU）。
- 优先级调度（`priority`）适合有 VIP 用户分层的场景。

### 4.2 [Chunked Prefill（分块预填充）](docs/4-2-chunked-prefill.md)

Chunked Prefill 是 vLLM 的重要性能特性：将长 prompt 的预填充分成多个小块，与正在生成的请求交错执行，从而减少长 prompt 导致的延迟峰值。

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--enable-chunked-prefill` | 启用分块预填充 | None（V1 默认启用） | **平台管理** |
| `--max-num-partial-prefills` | 最大并发部分预填充序列数 | 1 | **平台管理** |
| `--max-long-partial-prefills` | 长 prompt 最大并发预填充数 | 1 | **平台管理** |
| `--long-prefill-token-threshold` | "长 prompt"的 token 阈值 | 0 | **平台管理** |
| `--disable-chunked-mm-input` | 禁止将多模态输入分块调度 | `False` | **平台管理** |

### 4.3 [其他调度选项](docs/4-3-other-scheduling.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--stream-interval` | 流式输出的 token 缓冲大小。1 = 逐 token 发送（最平滑），更大值可减少开销 | 1 | **平台管理** |
| `--async-scheduling` | 异步调度，减少 GPU 空闲间隙，提升延迟和吞吐量 | `False` | **平台管理** |
| `--num-lookahead-slots` | 投机解码预留的 slot 数量 | 0 | **平台管理** |
| `--scheduler-cls` | 自定义调度器类 | None | **平台管理** |
| `--disable-hybrid-kv-cache-manager` | 为所有 attention 层分配相同大小的 KV Cache | `False` | **平台管理** |

---

## 5. 并行配置（ParallelConfig）

> 当单张 GPU 无法容纳大模型时，需要多 GPU 甚至多机并行推理。这是部署大型模型的关键配置。

### 5.1 [核心并行参数](docs/5-1-core-parallel.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--tensor-parallel-size` / `-tp` | **张量并行数**。将模型的每一层拆分到多张 GPU 上。例如 `-tp 4` 表示用 4 张 GPU 分担一个模型 | 1 | **普通用户** |
| `--pipeline-parallel-size` / `-pp` | **流水线并行数**。将模型的不同层分配到不同 GPU 上。适合显存受限但 GPU 数量多的场景 | 1 | **高级用户** |
| `--data-parallel-size` / `-dp` | **数据并行数**。运行多个模型副本处理不同请求，线性扩展吞吐量 | 1 | **高级用户** |

**产品设计建议：**
- `-tp` 是最常用的并行方式。建议平台根据所选模型大小和可用 GPU 数量自动推荐值，同时允许用户手动调整。例如："70B 模型建议使用 4 张 A100-80G（-tp 4）"。
- `-dp` 适合需要高吞吐量的场景。在 UI 上可以表述为"部署副本数"的概念。
- `-pp` 技术性较强，建议仅在高级模式下暴露。

### 5.2 [数据并行高级配置](docs/5-2-data-parallel-advanced.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--data-parallel-size-local` / `-dpl` | 本节点上的数据并行副本数 | None | **平台管理** |
| `--data-parallel-address` / `-dpa` | 数据并行集群头节点地址 | None | **平台管理** |
| `--data-parallel-backend` / `-dpb` | 数据并行后端：`mp`（多进程）/ `ray` | `mp` | **平台管理** |
| `--data-parallel-rank` / `-dpn` | 当前实例的数据并行 rank | None | **平台管理** |
| `--data-parallel-rpc-port` / `-dpp` | 数据并行 RPC 通信端口 | None | **平台管理** |
| `--data-parallel-start-rank` / `-dpr` | 次要节点的起始 rank | None | **平台管理** |
| `--data-parallel-external-lb` / `-dpe` | 使用外部负载均衡器模式 | `False` | **平台管理** |
| `--data-parallel-hybrid-lb` / `-dph` | 使用混合负载均衡模式 | `False` | **平台管理** |
| `--disable-nccl-for-dp-synchronization` | 数据并行同步使用 Gloo 替代 NCCL | `False` | **平台管理** |

### 5.3 [分布式执行](docs/5-3-distributed-execution.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--distributed-executor-backend` | 分布式后端：`mp`（单机多卡）/ `ray`（多机）/ `uni`（单进程）/ `external_launcher` | 自动选择 | **平台管理** |
| `--nnodes` / `-n` | 多机推理时的节点数 | 1 | **平台管理** |
| `--node-rank` / `-r` | 当前节点的 rank | 0 | **平台管理** |
| `--master-addr` | 主节点地址 | `127.0.0.1` | **平台管理** |
| `--master-port` | 主节点端口 | 29501 | **平台管理** |
| `--max-parallel-loading-workers` | 模型加载时的最大并行 worker 数（防止 OOM） | None | **平台管理** |

### 5.4 [Expert Parallelism（专家并行，MoE 模型）](docs/5-4-expert-parallelism.md)

Expert Parallelism 专用于 Mixture-of-Experts (MoE) 模型（如 Mixtral、DeepSeek-V3 等），将不同的专家分布在不同 GPU 上。

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--enable-expert-parallel` | 对 MoE 层使用专家并行替代张量并行 | `False` | **平台管理** |
| `--expert-placement-strategy` | 专家放置策略：`linear`（连续放置）/ `round_robin`（轮询放置） | `linear` | **平台管理** |
| `--all2all-backend` | MoE 通信后端：`naive`/`pplx`/`deepep_high_throughput`/`deepep_low_latency` 等 | None | **平台管理** |
| `--enable-eplb` | 启用 Expert Parallelism 负载均衡 | `False` | **平台管理** |
| `--eplb-config` | EPLB 配置 JSON | 默认配置 | **平台管理** |

### 5.5 [Dual Batch Overlap (DBO)](docs/5-5-dual-batch-overlap.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--enable-dbo` | 启用双 batch 重叠执行 | `False` | **平台管理** |
| `--dbo-prefill-token-threshold` | Prefill batch 的 token 阈值 | 512 | **平台管理** |
| `--dbo-decode-token-threshold` | Decode batch 的 token 阈值 | 32 | **平台管理** |

### 5.6 [Context Parallelism](docs/5-6-context-parallelism.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--decode-context-parallel-size` / `-dcp` | Decode context 并行度 | 1 | **平台管理** |
| `--dcp-kv-cache-interleave-size` | DCP KV Cache 交错大小 | 1 | **平台管理** |

### 5.7 [其他并行选项](docs/5-7-other-parallel.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--worker-cls` | Worker 类名 | `auto` | **平台管理** |
| `--worker-extension-cls` | Worker 扩展类 | 空 | **平台管理** |
| `--disable-custom-all-reduce` | 禁用自定义 all-reduce，回退到 NCCL | `False` | **平台管理** |
| `--ray-workers-use-nsight` | Ray worker 使用 nsight 性能分析 | `False` | **平台管理** |
| `--enable-multimodal-encoder-data-parallel` | 多模态编码器数据并行 | — | **平台管理** |

---

## 6. 模型加载配置（LoadConfig）

> 控制模型权重如何从存储加载到 GPU。
> 详细文档：[模型加载配置](docs/6-load-config.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--load-format` | 权重加载格式。`auto`（推荐）自动选择 safetensors 或 pytorch 格式。其他选项包括 `gguf`（用于量化模型）、`bitsandbytes`、`tensorizer` 等 | `auto` | **高级用户** |
| `--download-dir` | 模型下载和加载目录，默认使用 HuggingFace 缓存目录 | None | **平台管理** |
| `--ignore-patterns` | 加载时忽略的文件模式 | `['original/**/*']` | **平台管理** |
| `--model-loader-extra-config` | 模型加载器额外配置 | `{}` | **平台管理** |
| `--pt-load-map-location` | PyTorch checkpoint 的设备映射 | `cpu` | **平台管理** |
| `--safetensors-load-strategy` | Safetensors 加载策略：`lazy`（内存映射，推荐本地存储）/ `eager`（全量加载，推荐网络存储）/ `torchao` | `lazy` | **平台管理** |
| `--use-tqdm-on-load` | 加载时显示进度条 | `True` | **平台管理** |

**产品设计建议：**
- `--load-format` 对于使用 GGUF 格式量化模型的高级用户有用。
- `--download-dir` 由平台统一管理存储路径。
- `--safetensors-load-strategy` 应由平台根据存储类型自动设置（本地 SSD 用 `lazy`，NFS/对象存储用 `eager`）。

---

## 7. LoRA 配置（LoRAConfig）

> LoRA（Low-Rank Adaptation）是当前最流行的模型微调方法。vLLM 支持在推理时动态加载 LoRA 适配器，让一个基础模型同时服务多个微调版本。
> 详细文档：[LoRA 配置](docs/7-lora-config.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--enable-lora` | **启用 LoRA 支持**。开启后可以在推理时动态加载和切换 LoRA 适配器 | None | **高级用户** |
| `--max-loras` | 单个 batch 中最大 LoRA 数量。即同时可以用多少个不同的 LoRA | 1 | **高级用户** |
| `--max-lora-rank` | 最大 LoRA 秩：`1`/`8`/`16`/`32`/`64`/`128`/`256`/`320`/`512` | 16 | **高级用户** |
| `--max-cpu-loras` | CPU 内存中最大 LoRA 数量（需 >= `max-loras`） | None | **平台管理** |
| `--lora-dtype` | LoRA 数据类型：`auto`/`float16`/`bfloat16` | `auto` | **平台管理** |
| `--fully-sharded-loras` | 完全分片 LoRA 计算（高并发/高 rank 场景可能更快） | `False` | **平台管理** |
| `--lora-extra-vocab-size` | [已废弃] LoRA 额外词表大小 | 256 | **平台管理** |
| `--default-mm-loras` | 多模态默认 LoRA 映射 | None | **平台管理** |

**产品设计建议：**
- LoRA 是一个重要的高级特性。建议在 UI 上提供"启用微调适配器"开关，并允许用户上传或选择 LoRA 文件。
- `--max-loras` 和 `--max-lora-rank` 可以合并为"LoRA 配置"面板，仅对启用了 LoRA 的用户展示。

---

## 8. 多模态配置（MultiModalConfig）

> 控制多模态模型（如视觉语言模型）处理图片、视频等非文本输入的行为。

### 8.1 [核心多模态参数](docs/8-1-core-multimodal.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--limit-mm-per-prompt` | **每个 prompt 允许的多模态输入数量上限**。如 `{"image": 16, "video": 2}`。可以限制单次请求的图片/视频数量 | 各模态默认 999 | **高级用户** |
| `--media-io-kwargs` | 媒体处理额外参数，如视频帧数：`{"video": {"num_frames": 40}}` | `{}` | **高级用户** |

### 8.2 [处理器缓存](docs/8-2-processor-cache.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--mm-processor-cache-gb` | 多模态处理器缓存大小（GiB）。避免重复处理相同的图片/视频 | 4 | **平台管理** |
| `--mm-processor-cache-type` | 缓存类型：`lru`（LRU 缓存）/ `shm`（共享内存 FIFO） | `lru` | **平台管理** |
| `--mm-shm-cache-max-object-size-mb` | 共享内存缓存单对象最大大小（MiB） | 128 | **平台管理** |
| `--disable-mm-preprocessor-cache` | 禁用多模态预处理器缓存 | — | **平台管理** |

### 8.3 [编码器与性能](docs/8-3-encoder-performance.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--mm-encoder-tp-mode` | 多模态编码器的并行模式：`weights`（权重拆分）/ `data`（数据拆分） | `weights` | **平台管理** |
| `--mm-encoder-attn-backend` | 多模态编码器 attention 后端（如 `FLASH_ATTN`） | None | **平台管理** |
| `--mm-processor-kwargs` | 传给模型 processor 的额外参数 | None | **平台管理** |
| `--skip-mm-profiling` | 跳过多模态内存 profiling（加速启动但需用户自行估算内存） | `False` | **平台管理** |
| `--video-pruning-rate` | 视频 token 剪枝率 [0, 1)，减少视频 token 数量以节省计算 | None | **平台管理** |

### 8.4 [Embeddings](docs/8-4-embeddings.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--enable-mm-embeds` | 允许直接传入多模态 embeddings（安全风险，仅限受信用户） | `False` | **平台管理** |
| `--interleave-mm-strings` | 启用多模态 prompt 完全交错支持 | `False` | **平台管理** |

---

## 9. 结构化输出配置（StructuredOutputsConfig）

> 控制模型按照特定格式（如 JSON Schema）生成输出。这对需要可靠解析模型输出的应用非常重要。
> 详细文档：[结构化输出配置](docs/9-structured-outputs.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--reasoning-parser` | Reasoning 内容解析器，用于解析思维链内容为 OpenAI API 格式 | 空 | **高级用户** |
| `--reasoning-parser-plugin` | 自定义 reasoning 解析器插件路径 | 空 | **平台管理** |
| `--guided-decoding-backend` | [已废弃] Guided decoding 后端 | None | **平台管理** |
| `--guided-decoding-disable-additional-properties` | [已废弃] 禁用额外属性 | None | **平台管理** |
| `--guided-decoding-disable-any-whitespace` | [已废弃] 禁用任意空白 | None | **平台管理** |
| `--guided-decoding-disable-fallback` | [已废弃] 禁用 fallback | None | **平台管理** |

**产品设计建议：**
- 结构化输出（JSON mode、Function Calling）对于构建可靠 AI 应用至关重要。不过这些大多通过 API 请求参数控制（如 `response_format`），此处的配置主要是引擎级别的默认设置。
- `--reasoning-parser` 对于支持 Chain-of-Thought 推理的模型（如 DeepSeek-R1）很有用。

---

## 10. 可观测性配置（ObservabilityConfig）

> 生产环境中的监控和追踪能力。
> 详细文档：[可观测性配置](docs/10-observability.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--otlp-traces-endpoint` | OpenTelemetry traces 发送目标 URL | None | **平台管理** |
| `--collect-detailed-traces` | 收集详细追踪的模块：`all`/`model`/`worker`（可能影响性能） | None | **平台管理** |
| `--show-hidden-metrics-for-version` | 显示指定版本以来已隐藏的废弃 Prometheus 指标 | None | **平台管理** |

**产品设计建议：**
- 可观测性配置完全由平台基础设施团队管理。平台应自动接入自己的 Prometheus/Grafana 和 OpenTelemetry 系统，无需用户干预。

---

## 11. 编译优化配置（CompilationConfig）

> 通过 `torch.compile` 和 CUDA Graph 优化推理性能。这些是深度性能调优选项。
> 详细文档：[编译优化配置](docs/11-compilation.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--compilation-config` / `-O` | 完整的编译配置 JSON。快捷方式：`-O.mode=3` | 默认配置 | **平台管理** |
| `--cudagraph-capture-sizes` | CUDA Graph 捕获的 batch 大小列表 | 自动推断 | **平台管理** |
| `--max-cudagraph-capture-size` | CUDA Graph 最大捕获大小 | 自动推断 | **平台管理** |
| `--cuda-graph-sizes` | [已废弃] 使用 `--cudagraph-capture-sizes` 替代 | None | **平台管理** |

**产品设计建议：**
- 编译优化配置非常底层，应完全由平台管理。平台可根据 GPU 型号和模型类型选择最优编译策略。

---

## 12. 全局配置（VllmConfig 与通用选项）

### 12.1 [通用选项](docs/12-1-general-options.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--config` | YAML 配置文件路径。可以将所有参数写入一个配置文件中 | None | **平台管理** |
| `--api-server-count` / `-asc` | API 服务进程数 | 1 | **平台管理** |
| `--disable-log-stats` | 禁用统计日志 | `False` | **平台管理** |
| `--enable-log-requests` | 启用请求日志 | `False` | **平台管理** |
| `--aggregate-engine-logging` | 数据并行时聚合日志 | `False` | **平台管理** |
| `--headless` | 无头模式（多节点数据并行场景） | `False` | **平台管理** |

### 12.2 [VllmConfig 顶层配置](docs/12-2-vllm-config.md)

| 参数 | 说明 | 默认值 | 暴露建议 |
|------|------|--------|----------|
| `--additional-config` | 特定平台的额外配置 | `{}` | **平台管理** |
| `--speculative-config` | **投机解码配置**。使用小模型预测大模型的输出来加速推理 | None | **高级用户** |
| `--structured-outputs-config` | 结构化输出引擎配置 | 默认配置 | **平台管理** |
| `--kv-transfer-config` | 分布式 KV Cache 传输配置 | None | **平台管理** |
| `--kv-events-config` | KV 事件发布配置 | None | **平台管理** |
| `--ec-transfer-config` | 分布式 EC Cache 传输配置 | None | **平台管理** |

**产品设计建议：**
- `--speculative-config`（投机解码）是一个强大的加速特性。它用一个小模型来"猜"大模型的下一个 token，猜对了就跳过大模型计算。可以为高级用户提供"启用加速推理"选项，并自动匹配合适的 draft model。

---

## 参数暴露总览

### 普通用户可见参数（UI 主界面）

这些参数是用户部署模型时**必须或经常需要**设置的，应在主界面上清晰展示：

| 参数 | 一句话描述 |
|------|-----------|
| `--model` | 选择要部署的模型 |
| `--served-model-name` | 给模型起一个自定义名称 |
| `--max-model-len` | 设置最大上下文长度 |
| `--tensor-parallel-size` | 选择使用几张 GPU |

### 高级用户参数（高级设置面板）

这些参数需要一定技术理解，适合在"高级设置"或"专家模式"中暴露：

| 参数 | 一句话描述 |
|------|-----------|
| `--dtype` | 模型精度（影响显存和质量） |
| `--quantization` | 量化方式（大幅节省显存） |
| `--gpu-memory-utilization` | GPU 显存利用率 |
| `--kv-cache-dtype` | KV Cache 精度 |
| `--enable-prefix-caching` | 前缀缓存（共享 system prompt 加速） |
| `--max-num-seqs` | 最大并发序列数 |
| `--max-num-batched-tokens` | 最大 batch token 数 |
| `--scheduling-policy` | 调度策略 |
| `--pipeline-parallel-size` | 流水线并行度 |
| `--data-parallel-size` | 数据并行度 |
| `--enable-lora` | 启用 LoRA 微调适配器 |
| `--max-loras` | 最大 LoRA 数量 |
| `--max-lora-rank` | 最大 LoRA 秩 |
| `--load-format` | 模型加载格式 |
| `--enable-auto-tool-choice` | 启用 Function Calling |
| `--tool-call-parser` | 工具调用解析器 |
| `--tool-server` | 外部工具服务器 |
| `--chat-template` | 自定义对话模板 |
| `--limit-mm-per-prompt` | 每 prompt 多模态输入上限 |
| `--media-io-kwargs` | 媒体处理参数 |
| `--reasoning-parser` | 推理内容解析器 |
| `--speculative-config` | 投机解码配置 |
| `--seed` | 随机种子 |
| `--revision` | 模型版本 |
| `--generation-config` | 生成配置来源 |
| `--override-generation-config` | 覆盖生成配置 |
| `--lora-modules` | 预加载 LoRA 模块 |
| `--task` | 模型任务类型 |

### 平台管理参数（不暴露给用户）

其余所有参数均建议由平台/运维团队统一管理，包括但不限于：

- **网络与安全**：host、port、SSL、API key、CORS 等
- **基础设施**：分布式通信地址、端口、节点配置等
- **存储路径**：模型下载目录、日志配置等
- **性能调优**：编译配置、CUDA Graph、block size 等
- **监控追踪**：OpenTelemetry、Prometheus 指标等
- **安全开关**：trust-remote-code、enable-prompt-embeds 等

---

## 平台设计建议总结

### 1. 用户界面分层

```
┌─────────────────────────────────────────┐
│           基础配置（所有用户）              │
│  ┌─────────────┐  ┌──────────────────┐  │
│  │  模型选择     │  │  上下文长度       │  │
│  │  (model)     │  │  (max-model-len) │  │
│  └─────────────┘  └──────────────────┘  │
│  ┌─────────────┐  ┌──────────────────┐  │
│  │  模型名称     │  │  GPU 数量        │  │
│  │  (served-    │  │  (tensor-        │  │
│  │   model-name)│  │   parallel-size) │  │
│  └─────────────┘  └──────────────────┘  │
├─────────────────────────────────────────┤
│  ▼ 高级设置（展开/收起）                   │
│                                         │
│  性能调优: 量化 / 精度 / 显存利用率         │
│  调度策略: 并发数 / 调度模式                │
│  LoRA: 启用/配置适配器                     │
│  工具调用: Function Calling 配置           │
│  多模态: 图片/视频处理限制                  │
│  推理优化: 投机解码                        │
└─────────────────────────────────────────┘
```

### 2. 智能默认值

平台应根据上下文自动设置合理默认值：

- **根据模型自动设置**：`--max-model-len`、`--dtype`、`--trust-remote-code`、`--tool-call-parser`
- **根据硬件自动设置**：`--tensor-parallel-size`、`--gpu-memory-utilization`、`--block-size`
- **根据存储类型自动设置**：`--safetensors-load-strategy`、`--download-dir`
- **根据部署环境自动设置**：`--host`、`--port`、SSL 相关、分布式通信相关

### 3. 安全考量

以下参数涉及安全，平台必须谨慎处理：

| 参数 | 风险 | 建议 |
|------|------|------|
| `--trust-remote-code` | 执行不受信任的远程代码 | 仅对来自可信源的模型启用 |
| `--enable-prompt-embeds` | 可能通过恶意 embeddings 导致引擎崩溃 | 默认禁用 |
| `--enable-mm-embeds` | 同上，针对多模态 embeddings | 默认禁用 |
| `--allowed-local-media-path` | 可读取服务器文件系统 | 默认禁用 |
| `--hf-token` | HuggingFace 凭证泄露 | 通过 Secret 管理注入 |
| `--api-key` | API 认证 | 由平台认证系统管理 |
| `--trust-request-chat-template` | 注入恶意 prompt template | 默认禁用 |

---

## 附录：配置文件方式

vLLM 支持通过 YAML 配置文件传入所有参数，使用 `--config` 指定路径：

```yaml
# example-config.yaml
model: "meta-llama/Llama-3.1-70B-Instruct"
served-model-name: "my-llama"
max-model-len: 8192
tensor-parallel-size: 4
gpu-memory-utilization: 0.9
enable-prefix-caching: true
dtype: auto
```

启动命令：
```bash
vllm serve --config example-config.yaml
```

**平台建议**：用户在 UI 上的所有选择最终生成一份 YAML 配置文件，平台在此基础上注入平台管理的参数（网络、安全、监控等），然后启动 vLLM 服务。

---

## 附录：JSON 参数传递语法

vLLM 支持灵活的 JSON 参数传递方式，对于复杂的嵌套配置非常方便：

```bash
# 完整 JSON
--speculative-config '{"method": "draft_model", "draft_model": "small-model"}'

# 等价的展开写法
--speculative-config.method draft_model --speculative-config.draft_model small-model

# 列表追加
--cudagraph-capture-sizes+ 1 --cudagraph-capture-sizes+='2,4,8'
```

---

> 本文档基于 vLLM v0.11.2 生成，后续版本可能会新增或废弃部分参数。
> 标记为 `[DEPRECATED]` 的参数将在未来版本中移除，不建议在新项目中使用。
