# LiteLLM Proxy — K8s/OrbStack 部署指南

使用 LiteLLM Proxy 将 Claude 系列模型的 API 转发到本地，统一以 OpenAI 兼容接口对外暴露。本目录提供一键部署的 Kubernetes 配置文件，适用于 OrbStack 或标准 K8s 集群。

## 文件说明

```
litellm-proxy/
└── litellm.yaml    # 包含三个 K8s 资源的完整配置
```

`litellm.yaml` 包含三个 Kubernetes 资源（以 `---` 分隔）：

### 1. ConfigMap — LiteLLM 应用配置

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: litellm-config
  namespace: litellm
```

ConfigMap 存储 LiteLLM 的运行时配置，挂载到容器内的 `/app/config.yaml`。内容分三部分：

#### model_list — 模型路由表

定义对外暴露的模型名称，以及对应的上游 API 参数：

| 字段 | 说明 |
|------|------|
| `model_name` | 对外暴露的模型 ID，客户端调用时使用此名称 |
| `litellm_params.model` | LiteLLM 内部路由格式：`{provider}/{前缀}/{model_id}` |
| `litellm_params.api_base` | 上游 API 地址，`os.environ/` 前缀表示从环境变量读取 |
| `litellm_params.api_key` | 上游 API 鉴权 token，同样从环境变量读取 |

目前配置了以下模型：

**Anthropic 系列**（`anthropic/` 前缀，走 Anthropic API）：
- `claude-opus-4-6`
- `claude-sonnet-4-6`
- `claude-haiku-4-5-20251001`

**OpenAI 兼容系列**（`openai/` 前缀，走 OpenAI 兼容 API）：
- `glm-5`
- `kimi-k2-5`

#### general_settings — 通用设置

| 字段 | 说明 |
|------|------|
| `master_key` | LiteLLM Proxy 的访问密钥，客户端请求时需在 `Authorization` 头携带 |

#### litellm_settings — 行为设置

| 字段 | 默认值 | 说明 |
|------|--------|------|
| `drop_params` | `true` | 自动丢弃上游不支持的参数，避免报错 |
| `num_retries` | `2` | 请求失败时自动重试次数 |
| `use_chat_completions_url_for_anthropic_messages` | `true` | 让 `openai/` 前缀的模型通过 `/v1/chat/completions` 转发，而非 OpenAI Responses API。**非 Anthropic 模型（如 GLM、Kimi）使用 `/v1/messages` 端点时必须开启**，否则响应内容为空 |

---

### 2. Deployment — 应用部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: litellm
  namespace: litellm
```

| 配置项 | 值 | 说明 |
|--------|----|------|
| `replicas` | `1` | 单副本部署，按需调整 |
| `image` | `litellm/litellm:v1.82.6-nightly` | LiteLLM 官方镜像，固定版本 |
| `args` | `--config /app/config.yaml --port 4000` | 启动参数，指定配置文件和监听端口 |
| `for-restart: "1"` | Pod label | 用于 `kubectl rollout restart` 触发滚动更新 |

**环境变量（需修改为真实值）：**

| 环境变量 | 示例值 | 说明 |
|----------|--------|------|
| `LITELLM_LOG` | `DEBUG` | 日志级别，可选 `DEBUG` / `INFO`，调试时设为 `DEBUG` |
| `LITELLM_MASTER_KEY` | `sk-litellm-password` | Proxy 访问密钥，**部署前务必修改** |
| `ANTHROPIC_BASE_URL` | `https://your-api-provider.com/anthropic` | Anthropic 系列模型的上游 API 地址 |
| `ANTHROPIC_AUTH_TOKEN` | `sk-your-api-key` | Anthropic 系列模型的鉴权 token |
| `OPENAI_BASE_URL` | `https://your-openai-provider.com/v1` | OpenAI 兼容模型的上游 API 地址（供 glm-5、kimi-k2-5 等使用） |
| `OPENAI_API_KEY` | `sk-your-openai-key` | OpenAI 兼容模型的鉴权 token |

---

### 3. Service — 服务暴露

```yaml
apiVersion: v1
kind: Service
metadata:
  name: litellm
  namespace: litellm
spec:
  type: NodePort
```

| 字段 | 值 | 说明 |
|------|----|------|
| `type` | `NodePort` | 通过节点端口暴露，OrbStack 下可直接访问 |
| `port` / `targetPort` | `4000` | 容器内监听端口 |
| `nodePort` | `30040` | 宿主机访问端口 |

在 OrbStack 环境下，服务通过 `http://localhost:30040` 或 `http://<node-ip>:30040` 访问。

---

## 部署步骤

### 前置条件

- 已安装并启动 [OrbStack](https://orbstack.dev/)（或其他 K8s 集群）
- `kubectl` 已配置并连接到目标集群

### 1. 修改配置

编辑 `litellm.yaml`，将 Deployment 中的环境变量替换为真实值：

```yaml
env:
  - name: LITELLM_LOG
    value: "INFO"                         # 生产环境建议改为 INFO
  - name: LITELLM_MASTER_KEY
    value: "sk-your-custom-password"      # 自定义访问密钥
  - name: ANTHROPIC_BASE_URL
    value: "https://your-api-provider.com/anthropic"  # Anthropic 中转 API 地址
  - name: ANTHROPIC_AUTH_TOKEN
    value: "sk-your-anthropic-key"        # Anthropic API Key
  - name: OPENAI_BASE_URL
    value: "https://your-openai-provider.com/v1"      # OpenAI 兼容 API 地址
  - name: OPENAI_API_KEY
    value: "sk-your-openai-key"           # OpenAI 兼容 API Key
```

### 2. 创建 Namespace

```bash
kubectl create namespace litellm
```

### 3. 部署所有资源

```bash
kubectl apply -f litellm.yaml
```

### 4. 验证部署状态

```bash
# 查看 Pod 状态
kubectl get pods -n litellm

# 查看服务
kubectl get svc -n litellm

# 查看日志
kubectl logs -n litellm -l app=litellm -f
```

### 5. 测试接口

```bash
curl http://localhost:30040/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-litellm-password" \
  -d '{
    "model": "claude-sonnet-4-6",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

---

## 更新配置

修改 `litellm.yaml` 后重新应用：

```bash
kubectl apply -f litellm.yaml

# 触发 Pod 滚动重启（使 ConfigMap 变更生效）
kubectl rollout restart deployment/litellm -n litellm
```

---

## 在 Claude Code 中使用

部署完成后，可将 LiteLLM Proxy 配置为 Claude Code 的 API 中转：

```bash
# 设置环境变量
export ANTHROPIC_BASE_URL=http://localhost:30040
export ANTHROPIC_AUTH_TOKEN=sk-litellm-password

# 启动 Claude Code
claude
```

或写入 shell 配置文件（`~/.zshrc` / `~/.bashrc`）中永久生效。

---

## 相关资源

- [LiteLLM 官方文档](https://docs.litellm.ai/)
- [LiteLLM Docker Hub](https://hub.docker.com/r/litellm/litellm)
- [OrbStack 官网](https://orbstack.dev/)
- [Claude Code 中转 API 使用指南](../claude-code-usage/)
