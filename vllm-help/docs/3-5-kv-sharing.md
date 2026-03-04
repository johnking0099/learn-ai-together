# 3.5 KV Sharing

> 返回 [README 总览](../README.md)

KV Sharing 是一种实验性功能，允许在特定架构（如 YOCO）中共享 KV Cache，以减少显存占用并加速预填充过程。

---

## `--kv-sharing-fast-prefill`

| 属性 | 值 |
|------|-----|
| **说明** | 实验性功能：在支持 KV Sharing 的模型架构（如 YOCO — You Only Cache Once）中启用快速预填充。YOCO 架构的核心思想是只缓存一次 KV，然后在多层之间共享，从而大幅减少 KV Cache 的显存占用。启用此选项可以进一步优化预填充阶段的计算效率 |
| **类型** | bool |
| **默认值** | False |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve microsoft/YOCO-model --kv-sharing-fast-prefill
```
> 在部署 YOCO 架构模型时启用快速预填充，利用 KV 共享机制减少预填充阶段的重复计算，提升长上下文场景下的首 token 响应速度。

---
