# 2.4 SSL/TLS 安全

> 返回 [README 总览](../README.md)

配置 API 服务器的 HTTPS/TLS 加密通信。在生产环境中，启用 SSL/TLS 是保护数据传输安全的基本要求。

---

## `--ssl-certfile`

| 属性 | 值 |
|------|-----|
| **说明** | SSL/TLS 证书文件的路径（PEM 格式）。该证书用于向客户端证明服务器身份，并建立加密连接。通常为 CA 签发的证书或自签名证书 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --ssl-certfile /etc/ssl/certs/vllm.pem --ssl-keyfile /etc/ssl/private/vllm-key.pem
```
> 使用指定的 SSL 证书和私钥启动 HTTPS 服务。客户端可通过 `https://` 协议安全访问 API。

---

## `--ssl-keyfile`

| 属性 | 值 |
|------|-----|
| **说明** | SSL/TLS 私钥文件的路径（PEM 格式）。私钥与证书配对使用，用于加密通信。私钥文件应严格保护，仅允许服务进程读取 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --ssl-certfile /etc/ssl/certs/vllm.pem --ssl-keyfile /etc/ssl/private/vllm-key.pem
```
> 指定与 SSL 证书配对的私钥文件。私钥文件权限建议设置为 600，仅允许属主读写。

---

## `--ssl-ca-certs`

| 属性 | 值 |
|------|-----|
| **说明** | CA（证书颁发机构）证书文件路径。用于验证客户端证书的信任链。在双向 TLS（mTLS）场景中，服务器通过此 CA 证书验证客户端身份 |
| **类型** | string |
| **默认值** | None |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --ssl-certfile /etc/ssl/certs/vllm.pem --ssl-keyfile /etc/ssl/private/vllm-key.pem --ssl-ca-certs /etc/ssl/certs/ca.pem --ssl-cert-reqs 2
```
> 配置双向 TLS 认证。服务器使用 CA 证书验证客户端证书，确保只有持有合法证书的客户端才能连接。

---

## `--ssl-cert-reqs`

| 属性 | 值 |
|------|-----|
| **说明** | 客户端证书验证要求的级别。`0` 表示不要求客户端证书（单向 TLS），`1` 表示可选验证（客户端可以不提供证书），`2` 表示必须提供有效客户端证书（双向 TLS / mTLS） |
| **类型** | int |
| **默认值** | 0 |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --ssl-certfile /etc/ssl/certs/vllm.pem --ssl-keyfile /etc/ssl/private/vllm-key.pem --ssl-cert-reqs 2 --ssl-ca-certs /etc/ssl/certs/ca.pem
```
> 要求客户端必须提供有效证书（mTLS）。适用于对安全要求极高的内部服务通信，确保双方身份互相验证。

---

## `--enable-ssl-refresh`

| 属性 | 值 |
|------|-----|
| **说明** | 启用 SSL 证书文件的自动刷新功能。当证书文件发生变更时（例如证书续期），服务会自动加载新证书，无需重启。适用于使用自动证书管理工具（如 cert-manager）的场景 |
| **类型** | bool |
| **默认值** | `False` |
| **暴露建议** | 平台管理 |

**示例：**
```bash
vllm serve --ssl-certfile /etc/ssl/certs/vllm.pem --ssl-keyfile /etc/ssl/private/vllm-key.pem --enable-ssl-refresh
```
> 启用 SSL 证书自动刷新。当 cert-manager 等工具更新证书文件后，vLLM 会自动加载新证书，保证服务不中断。

---
