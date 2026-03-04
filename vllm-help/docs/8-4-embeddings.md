# 8.4 Embeddings

> 返回 [README 总览](../README.md)

多模态 Embeddings 相关配置，包括直接传入多模态嵌入向量和多模态 prompt 交错支持。

---

## `--enable-mm-embeds`

| 属性 | 值 |
|------|-----|
| **说明** | 允许 API 请求中直接传入预计算的多模态 embedding 向量，而非原始图片/视频数据。启用后，客户端可以在自己的环境中完成视觉编码，然后将 embedding 向量发送给 vLLM 进行后续推理。注意：此选项存在安全风险，恶意构造的 embedding 可能导致引擎异常，应仅对受信任的客户端启用 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --enable-mm-embeds
```
> 启用多模态 embedding 直传功能，允许客户端发送预计算好的视觉 embedding 向量，适合视觉编码已在客户端完成的架构。

---

## `--interleave-mm-strings`

| 属性 | 值 |
|------|-----|
| **说明** | 启用多模态 prompt 的完全交错支持。开启后，prompt 中的文本和多模态内容（图片、视频等）可以自由交错排列，而不要求所有多模态输入集中在 prompt 的特定位置。这对于需要在对话中多处插入图片的复杂多模态交互场景非常有用 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --interleave-mm-strings
```
> 启用多模态 prompt 交错模式，允许在对话的任意位置插入图片或视频，而非仅在 prompt 开头或固定位置。

---
