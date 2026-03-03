# 8.2 处理器缓存

> 返回 [README 总览](../README.md)

多模态处理器缓存配置，用于避免重复处理相同的图片/视频输入，提升多模态推理的效率。

---

## `--mm-processor-cache-gb`

| 属性 | 值 |
|------|-----|
| **说明** | 多模态处理器缓存的大小，单位为 GiB。当同一张图片或视频在多次请求中反复出现时，缓存可以避免重复的预处理计算（如图片 resize、视频抽帧等），直接复用已处理的结果，显著提升推理效率 |
| **类型** | float |
| **默认值** | 4 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --mm-processor-cache-gb 8
```
> 将多模态处理器缓存设置为 8 GiB，适合图片重复率较高的场景（如电商商品图片问答）。

---

## `--mm-processor-cache-type`

| 属性 | 值 |
|------|-----|
| **说明** | 多模态处理器缓存的实现类型。`lru` 使用进程内的 LRU（Least Recently Used）缓存，适合单进程场景；`shm` 使用共享内存 FIFO 缓存，适合多进程或数据并行场景下跨进程共享缓存 |
| **类型** | enum |
| **可选值** | `lru` / `shm` |
| **默认值** | `lru` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --mm-processor-cache-type shm
```
> 使用共享内存缓存，适合多进程数据并行部署场景，多个 worker 之间可以共享预处理结果。

---

## `--mm-shm-cache-max-object-size-mb`

| 属性 | 值 |
|------|-----|
| **说明** | 当缓存类型为 `shm`（共享内存）时，单个缓存对象允许的最大大小，单位为 MiB。超过此大小的处理结果将不会被缓存。需要根据实际多模态输入的大小进行调整，例如高分辨率图片或长视频的处理结果可能需要更大的上限 |
| **类型** | int |
| **默认值** | 128 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --mm-processor-cache-type shm --mm-shm-cache-max-object-size-mb 256
```
> 将共享内存缓存的单对象上限设置为 256 MiB，以支持缓存高分辨率图片的处理结果。

---

## `--disable-mm-preprocessor-cache`

| 属性 | 值 |
|------|-----|
| **说明** | 禁用多模态预处理器缓存。设置此标志后，每次请求中的图片/视频都会重新执行完整的预处理流程，不再复用之前的处理结果。通常仅在调试预处理逻辑或排查缓存一致性问题时使用 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --disable-mm-preprocessor-cache
```
> 关闭多模态预处理器缓存，用于调试场景以确保每次请求都执行完整的预处理流程。

---
