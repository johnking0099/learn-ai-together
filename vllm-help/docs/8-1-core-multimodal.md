# 8.1 核心多模态参数

> 返回 [README 总览](../README.md)

控制多模态模型（如视觉语言模型 LLaVA、Qwen-VL 等）处理图片、视频等非文本输入时的核心行为参数。

---

## `--limit-mm-per-prompt`

| 属性 | 值 |
|------|-----|
| **说明** | 每个 prompt 允许的多模态输入数量上限。以 JSON 字典形式指定每种模态的最大数量，例如 `{"image": 16, "video": 2}` 表示单个 prompt 最多包含 16 张图片和 2 个视频。用于防止用户在单次请求中传入过多媒体文件，避免 GPU 显存溢出或处理超时 |
| **类型** | string (JSON dict) |
| **默认值** | 各模态默认 999 |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve meta-llama/Llama-4-Scout-17B-16E-Instruct --limit-mm-per-prompt '{"image": 8, "video": 1}'
```
> 限制每个 prompt 最多包含 8 张图片和 1 个视频，适合对资源消耗有严格控制要求的生产环境。

---

## `--media-io-kwargs`

| 属性 | 值 |
|------|-----|
| **说明** | 媒体 I/O 处理的额外参数，以 JSON 字典形式传入。可以针对不同模态（如 video）设置解码参数，例如指定视频抽帧数量。这些参数会传递给底层的媒体处理模块，用于精细控制多模态输入的预处理行为 |
| **类型** | string (JSON dict) |
| **默认值** | `{}` |
| **暴露建议** | 高级用户 |

**示例：**
```bash
vllm serve Qwen/Qwen2.5-VL-7B-Instruct --media-io-kwargs '{"video": {"num_frames": 40}}'
```
> 将视频输入的抽帧数设置为 40 帧，适合需要更高时间分辨率的视频理解任务。

---
