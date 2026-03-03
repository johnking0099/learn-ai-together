# 2.1 网络与访问

> 返回 [README 总览](../README.md)

控制 vLLM API 服务器的网络监听地址、端口以及访问认证。这些配置通常由平台基础设施团队统一管理。

---

## `--host`

| 属性 | 值 |
|------|-----|
| **说明** | 设置 API 服务器监听的主机地址。可以使用 IP 地址或主机名。设为 `0.0.0.0` 则监听所有网络接口，设为 `127.0.0.1` 则仅允许本机访问 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --host 0.0.0.0
```
> 监听所有网络接口，允许外部网络访问 API 服务。适用于容器化部署或需要远程访问的场景。

---

## `--port`

| 属性 | 值 |
|------|-----|
| **说明** | 设置 API 服务器监听的端口号。客户端通过此端口发送请求 |
| **类型** | int |
| **默认值** | 8000 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --port 8080
```
> 将 API 服务端口改为 8080。当默认端口 8000 被其他服务占用时，可通过此参数指定其他端口。

---

## `--uds`

| 属性 | 值 |
|------|-----|
| **说明** | 指定 Unix Domain Socket 文件路径。使用 UDS 时，服务通过文件系统 socket 通信而非 TCP，性能更好且更安全。设置后会忽略 `--host` 和 `--port` 参数 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --uds /tmp/vllm.sock
```
> 通过 Unix Domain Socket 提供服务。适用于同一台机器上的服务间通信，如 Nginx 反向代理到 vLLM。

---

## `--root-path`

| 属性 | 值 |
|------|-----|
| **说明** | 设置 FastAPI 的 root_path，用于反向代理场景。当 vLLM 部署在反向代理（如 Nginx、Traefik）后面，且代理添加了路径前缀时，需要设置此参数以确保 API 路由正确 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --root-path /api/v1/llm
```
> 设置路径前缀为 `/api/v1/llm`。当 Nginx 将 `/api/v1/llm/*` 的请求代理到 vLLM 时，vLLM 能正确处理路由。

---

## `--api-key`

| 属性 | 值 |
|------|-----|
| **说明** | 设置 API 访问密钥。配置后，所有请求必须在 HTTP Header 中携带 `Authorization: Bearer <api-key>` 才能访问服务。用于保护 API 免受未授权访问 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --api-key sk-my-secret-key-12345
```
> 启用 API 密钥认证。客户端调用时需携带对应密钥：`curl -H "Authorization: Bearer sk-my-secret-key-12345" http://localhost:8000/v1/chat/completions`。

---
