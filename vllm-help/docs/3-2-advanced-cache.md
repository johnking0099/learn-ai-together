# 3.2 高级缓存参数

> 返回 [README 总览](../README.md)

高级缓存参数用于精细调控 KV Cache 的内存分配和行为，包括手动指定缓存大小、交换空间、CPU 卸载等。

---

## `--kv-cache-memory-bytes`

| 属性 | 值 |
|------|-----|
| **说明** | 手动指定每张 GPU 上 KV Cache 占用的内存大小（字节）。相比 `--gpu-memory-utilization` 的百分比方式，此参数提供了更精细的绝对值控制，适用于需要精确管理显存分配的场景 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --kv-cache-memory-bytes 8589934592
```
> 为每张 GPU 分配 8 GiB 的 KV Cache 空间，精确控制缓存大小，避免与其他显存分配发生冲突。

---

## `--block-size`

| 属性 | 值 |
|------|-----|
| **说明** | KV Cache 缓存块大小，以 token 数为单位。vLLM 以 block 为粒度管理 KV Cache 内存。更大的 block 可以减少管理开销但可能浪费更多碎片空间。CUDA 平台上最大支持 32 |
| **类型** | enum |
| **可选值** | `1` / `8` / `16` / `32` / `64` / `128` / `256` |
| **默认值** | 自动推断 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --block-size 16
```
> 将缓存块大小设为 16 个 token。较小的 block size 可减少内存碎片但增加管理开销，适合短请求较多的场景。

---

## `--swap-space`

| 属性 | 值 |
|------|-----|
| **说明** | 每张 GPU 对应的 CPU 交换空间大小（GiB）。当 GPU 显存中的 KV Cache 不足时，vLLM 会将暂时不活跃的请求的 KV Cache 交换到 CPU 内存中，待有空间后再换回 |
| **类型** | float |
| **默认值** | 4 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --swap-space 8
```
> 将交换空间增大到 8 GiB，为高并发场景提供更多缓冲，减少因显存不足而拒绝请求的概率。

---

## `--cpu-offload-gb`

| 属性 | 值 |
|------|-----|
| **说明** | 每张 GPU 卸载到 CPU 内存的空间大小（GiB）。相当于虚拟扩展 GPU 显存，将部分模型权重或 KV Cache 存放在 CPU 内存中。需要高速 CPU-GPU 互联（如 NVLink）才能获得较好性能 |
| **类型** | float |
| **默认值** | 0 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --cpu-offload-gb 4
```
> 将 4 GiB 的数据卸载到 CPU 内存，在 GPU 显存紧张时扩展可用内存空间，代价是部分数据访问速度变慢。

---

## `--num-gpu-blocks-override`

| 属性 | 值 |
|------|-----|
| **说明** | 手动覆盖 GPU KV Cache blocks 的数量。主要用于测试和调试目的，生产环境中一般不需要手动设置此值 |
| **类型** | int |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --num-gpu-blocks-override 2048
```
> 强制设置 GPU blocks 数量为 2048，用于测试特定内存配置下的服务行为。

---

## `--calculate-kv-scales`

| 属性 | 值 |
|------|-----|
| **说明** | 当 KV Cache 使用 fp8 数据类型时，是否动态计算缩放因子（scaling factors）。启用后可以改善 fp8 KV Cache 的精度表现，但会增加少量计算开销 |
| **类型** | bool |
| **默认值** | False |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --kv-cache-dtype fp8 --calculate-kv-scales
```
> 搭配 fp8 KV Cache 使用，动态计算缩放因子以获得更好的精度，适合对生成质量要求较高的场景。

---

## `--prefix-caching-hash-algo`

| 属性 | 值 |
|------|-----|
| **说明** | 前缀缓存使用的哈希算法。用于计算 prompt 前缀的哈希值以判断是否可以复用缓存。`sha256` 是标准选择，`sha256_cbor` 使用 CBOR 序列化后再哈希 |
| **类型** | enum |
| **可选值** | `sha256` / `sha256_cbor` |
| **默认值** | sha256 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --enable-prefix-caching --prefix-caching-hash-algo sha256
```
> 使用 SHA-256 哈希算法进行前缀缓存匹配，这是默认且推荐的选择，具有良好的碰撞抗性和计算效率。

---
