# 2.2 CORS（跨域资源共享）

> 返回 [README 总览](../README.md)

控制 API 服务器的跨域资源共享策略。当前端应用需要直接从浏览器访问 vLLM API 时，需要正确配置 CORS。

---

## `--allowed-origins`

| 属性 | 值 |
|------|-----|
| **说明** | 设置允许跨域请求的来源列表。浏览器会在跨域请求中检查 `Access-Control-Allow-Origin` 响应头，只有匹配的来源才被允许。使用 `*` 表示允许所有来源 |
| **类型** | string |
| **默认值** | `['*']` |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve --allowed-origins '["https://app.example.com", "https://dev.example.com"]'
```
> 仅允许来自 `https://app.example.com` 和 `https://dev.example.com` 的跨域请求。生产环境中应限制为具体域名，而非使用通配符 `*`。

---

## `--allowed-methods`

| 属性 | 值 |
|------|-----|
| **说明** | 设置允许的 HTTP 请求方法列表。控制跨域请求可以使用哪些 HTTP 方法（如 GET、POST、PUT 等）。使用 `*` 表示允许所有方法 |
| **类型** | string |
| **默认值** | `['*']` |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve --allowed-methods '["GET", "POST"]'
```
> 仅允许 GET 和 POST 方法的跨域请求。由于 vLLM API 主要使用 POST 方法，限制为 GET 和 POST 即可满足大多数场景。

---

## `--allowed-headers`

| 属性 | 值 |
|------|-----|
| **说明** | 设置允许的 HTTP 请求头列表。控制跨域请求可以携带哪些自定义头部（如 `Authorization`、`Content-Type` 等）。使用 `*` 表示允许所有头部 |
| **类型** | string |
| **默认值** | `['*']` |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve --allowed-headers '["Content-Type", "Authorization"]'
```
> 仅允许跨域请求携带 `Content-Type` 和 `Authorization` 头部。这是使用 API Key 认证时的最小必要配置。

---

## `--allow-credentials`

| 属性 | 值 |
|------|-----|
| **说明** | 是否允许跨域请求携带凭证（如 Cookies、HTTP 认证信息）。启用后，浏览器会在跨域请求中发送 Cookies 等凭证信息。注意：当此项为 `True` 时，`--allowed-origins` 不能为 `*` |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台决定 |

**示例：**
```bash
vllm serve --allow-credentials --allowed-origins '["https://app.example.com"]'
```
> 允许来自 `https://app.example.com` 的跨域请求携带 Cookie 等凭证。适用于前端应用通过 Cookie 进行用户认证的场景。

---
