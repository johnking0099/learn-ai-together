# 2.6 LoRA 模块加载（前端层面）

> 返回 [README 总览](../README.md)

在前端服务层面预加载 LoRA 模块。这允许在启动时就指定可用的 LoRA 适配器，用户可以在 API 请求中引用这些预加载的模块。

---

## `--lora-modules`

| 属性 | 值 |
|------|-----|
| **说明** | 预加载的 LoRA 适配器模块列表。支持 `name=path` 的简写格式或完整的 JSON 配置。在服务启动时即加载指定的 LoRA 模块，用户可以在 API 请求的 `model` 字段中使用 LoRA 名称来调用对应的微调模型。需要先通过 `--enable-lora` 启用 LoRA 支持 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 高级配置 |

**示例：**
```bash
vllm serve --enable-lora --lora-modules customer-service=/models/lora/customer-service legal-assistant=/models/lora/legal
```
> 预加载两个 LoRA 模块：`customer-service` 和 `legal-assistant`。用户在 API 请求中可以通过 `"model": "customer-service"` 来使用对应的微调模型进行推理。

---
