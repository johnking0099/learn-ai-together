# 10 可观测性配置（ObservabilityConfig）

> 返回 [README 总览](../README.md)

生产环境中的监控和追踪配置。通过 OpenTelemetry 和 Prometheus 指标实现对推理服务的可观测性。

---

## `--otlp-traces-endpoint`

| 属性 | 值 |
|------|-----|
| **说明** | OpenTelemetry traces 发送目标 URL。配置后 vLLM 会将分布式追踪数据发送到指定的 OTLP collector 端点，用于分析请求延迟、定位性能瓶颈和排查故障 |
| **类型** | string |
| **默认值** | None（不发送追踪数据） |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --otlp-traces-endpoint http://jaeger:4318/v1/traces
```
> 将 OpenTelemetry 追踪数据发送到 Jaeger 收集器。平台运维可以在 Jaeger UI 中查看每个推理请求的完整链路，包括调度、预填充、解码等各阶段的耗时。

---

## `--collect-detailed-traces`

| 属性 | 值 |
|------|-----|
| **说明** | 收集详细追踪的模块名称。可以指定 `all` 收集所有模块，或指定具体模块如 `model`、`worker` 等。启用详细追踪会增加额外开销，可能影响推理性能，建议仅在排查问题时临时开启 |
| **类型** | string |
| **默认值** | None（不收集详细追踪） |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --otlp-traces-endpoint http://jaeger:4318/v1/traces --collect-detailed-traces model
```
> 收集模型层面的详细追踪信息。这会在追踪数据中包含模型前向传播的细粒度耗时，帮助定位模型执行中的性能瓶颈。注意会带来额外性能开销。

---

## `--show-hidden-metrics-for-version`

| 属性 | 值 |
|------|-----|
| **说明** | 显示指定版本以来已隐藏的废弃 Prometheus 指标。vLLM 在版本迭代中会废弃旧指标并引入新指标，此参数可以临时恢复旧指标的暴露，用于兼容尚未迁移到新指标的监控面板 |
| **类型** | string |
| **默认值** | None（不显示已隐藏的指标） |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve meta-llama/Llama-3.1-8B --show-hidden-metrics-for-version 0.10.0
```
> 显示自 v0.10.0 以来被隐藏的废弃 Prometheus 指标。适用于升级 vLLM 版本后，Grafana 监控面板依赖旧指标名称尚未更新的过渡期。
