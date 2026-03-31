# OpenClaw 多 Gateway + 微信 Channel 部署指南

> 适用平台：macOS / Linux
> OpenClaw 版本：2026.3.23-2 及以上

---

## 目录

1. [背景与架构说明](#1-背景与架构说明)
2. [前置条件](#2-前置条件)
3. [安装第一个 Gateway（主实例）](#3-安装第一个-gateway主实例)
4. [安装第二个 Gateway（demo 实例）](#4-安装第二个-gatewaydemo-实例)
5. [安装微信 Channel 插件](#5-安装微信-channel-插件)
6. [修复微信插件加载失败](#6-修复微信插件加载失败)
7. [连接微信账号（扫码登录）](#7-连接微信账号扫码登录)
8. [端口与浏览器配置](#8-端口与浏览器配置)
9. [验证与日常运维](#9-验证与日常运维)
10. [常见问题](#10-常见问题)

---

## 1. 背景与架构说明

OpenClaw 支持在同一台服务器上运行**多个独立 Gateway 实例**，常见场景：

- **多账号隔离**：不同业务线用不同 Bot，互不干扰
- **救援 Bot**：当主 Bot 配置损坏时，用第二实例应急修复
- **多渠道隔离**：微信、Telegram 等渠道分布在不同实例，降低相互影响

每个实例必须完全隔离以下四要素：

| 隔离项 | 说明 |
|--------|------|
| `--profile <name>` | 决定配置文件路径和状态目录，最简单的隔离方式 |
| `gateway.port` | 每个实例必须使用不同的基础端口 |
| `OPENCLAW_STATE_DIR` | 会话、凭据、缓存的存储目录 |
| `agents.defaults.workspace` | 每个实例的 Agent 工作目录 |

> **端口间距建议**：相邻实例基础端口至少间隔 20，避免派生端口（browser/canvas/CDP）冲突。

---

## 2. 前置条件

```bash
# 确认 Node.js 版本 >= 20
node -v

# 确认 openclaw 已全局安装
openclaw --version

# 查看全局安装路径（后续修复插件需要）
npm root -g
# 示例输出：/Users/yourname/.nvm/versions/node/v24.14.0/lib/node_modules
```

---

## 3. 安装第一个 Gateway（主实例）

主实例不使用 `--profile` 参数，直接使用默认配置。

### 3.1 交互式初始化

```bash
openclaw onboard
```

关键选项参考：

| 配置项 | 建议值 |
|--------|--------|
| Setup mode | Manual |
| What to set up | Local gateway |
| Gateway port | `18789` |
| Gateway bind | Loopback (127.0.0.1) 或根据需要选 All interfaces |
| Gateway auth | Token |

### 3.2 安装为系统服务

```bash
# 安装 LaunchAgent（macOS）或 systemd service（Linux）
openclaw gateway install

# 启动
openclaw gateway start

# 查看状态
openclaw status
```

---

## 4. 安装第二个 Gateway（demo 实例）

### 4.1 交互式初始化

```bash
openclaw --profile demo onboard
```

> **注意：端口必须与主实例不同**，建议使用 `19789`（主实例 `18789` + 20 以上）。

### 4.2 手动填写 Workspace 路径

onboard 过程中会提示输入 Workspace 目录。**此路径需要用户手动输入，OpenClaw 自动填充的路径可能不对**，请确保输入正确：

```
◇  Workspace directory
│  /Users/yourname/.openclaw-demo/workspace
```

> **路径规则**：profile 名为 `demo` 时，对应的状态目录为 `~/.openclaw-demo/`，workspace 应填写为 `~/.openclaw-demo/workspace`。
> 请将 `yourname` 替换为你的实际用户名，**不要照抄示例路径**，否则数据目录会创建在错误位置。

其他关键选项：

| 配置项 | 建议值 |
|--------|--------|
| Gateway port | `19789`（与主实例不同） |
| Workspace directory | `~/.openclaw-demo/workspace`（手动输入） |

### 4.3 安装并启动

```bash
openclaw --profile demo gateway install
openclaw --profile demo gateway start

# 验证
openclaw --profile demo status
```

两个实例的配置文件分别存储在：

```
~/.openclaw/openclaw.json         # 主实例（无 --profile）
~/.openclaw-demo/openclaw.json    # demo 实例（--profile demo）
```

---

## 5. 安装微信 Channel 插件

微信 Channel 以插件形式提供，需单独安装。以下以 `--profile demo` 为例。

```bash
openclaw --profile demo plugins install @tencent-weixin/openclaw-weixin@latest
```

安装完成后输出类似：

```
Installed plugin: openclaw-weixin
Restart the gateway to load plugins.
```

> **安全提示**：安装时会显示 WARNING，提示插件含 `child_process` 调用和网络发送行为。这是微信 CLI 工具的正常行为，官方插件，可以接受。

---

## 6. 修复微信插件加载失败

### 6.1 问题现象

重启 Gateway 或执行 `openclaw --profile demo configure` 后，出现如下错误：

```
openclaw-weixin failed to load: Error: Cannot find module 'openclaw/plugin-sdk/channel-config-schema'
```

### 6.2 原因分析

插件的 `node_modules` 中没有 `openclaw` 这个依赖。插件安装时只安装了自身的依赖（`qrcode-terminal`、`zod`），但没有将宿主包 `openclaw` 链接进来。

### 6.3 修复步骤

```bash
# 1. 进入插件的 node_modules 目录
cd ~/.openclaw-demo/extensions/openclaw-weixin/node_modules

# 2. 创建 openclaw 的符号链接，指向全局安装路径
#    将 /Users/yourname/.nvm/... 替换为你实际的全局 node_modules 路径
ln -s /Users/yourname/.nvm/versions/node/v24.14.0/lib/node_modules/openclaw openclaw

# 验证
ls -la | grep openclaw
# 预期输出：openclaw -> /Users/yourname/.nvm/.../lib/node_modules/openclaw
```

**如何确认全局 openclaw 路径：**

```bash
# 方法一
npm root -g
# 输出示例：/Users/yourname/.nvm/versions/node/v24.14.0/lib/node_modules
# 则全局 openclaw 路径为：<上述路径>/openclaw

# 方法二
which openclaw | xargs readlink -f | xargs dirname | xargs dirname
```

### 6.4 重启 Gateway 使修复生效

```bash
openclaw --profile demo gateway restart
```

再次运行 configure，不再报错即修复成功：

```bash
openclaw --profile demo configure
```

---

## 7. 连接微信账号（扫码登录）

微信插件目前**不支持交互式向导配置**。通过 `openclaw --profile demo configure` 进入渠道配置时，选择 `openclaw-weixin` 后会看到：

```
◇  Channel setup ──────────────────────────────────────╮
│                                                      │
│  openclaw-weixin does not support guided setup yet.  │
│                                                      │
├──────────────────────────────────────────────────────╯
```

因此需要使用专用的登录命令完成连接：

```bash
openclaw --profile demo channels login --channel openclaw-weixin
```

执行后终端会展示二维码：

```
正在启动微信扫码登录...

使用微信扫描以下二维码，以完成连接：

▄▄▄▄▄▄▄▄▄▄▄▄▄...（二维码）

等待连接结果...

✅ 与微信连接成功！
```

> **如果终端二维码无法识别**，命令会同时输出一个可在浏览器打开的链接，用微信扫描该页面的二维码即可。

登录成功后，配置会自动写入 `~/.openclaw-demo/openclaw.json`，无需手动操作。

---

## 8. 端口与浏览器配置

### 端口自动分配规则

浏览器相关端口均基于 `gateway.port` 自动派生：

| 派生服务 | 计算规则 | 主实例（base=18789） | demo 实例（base=19789） |
|---------|---------|---------------------|----------------------|
| Gateway HTTP/WS | base | `18789` | `19789` |
| Browser control | base + 2 | `18791` | `19791` |
| CDP 端口池 | base+11 ~ base+110 | `18800`~`18899` | `19800`~`19899` |

两实例基础端口间距 ≥ 20 即可保证 CDP 端口池不重叠。用 `lsof` 检查关键端口是否被占用：

```bash
lsof -i :19789 -i :19791 -i :19800
```

### 浏览器 Profile 隔离（重要）

端口不冲突还不够——**每个 Gateway 实例还必须使用不同的浏览器 Profile 名**。

所有实例默认使用名为 `openclaw` 的浏览器 profile，对应同一个 Chrome 用户数据目录。Chrome 不允许同一目录被两个进程同时打开，会报：

```
Failed to start Chrome CDP on port <N> for profile "openclaw"
```

**解决方法：** 通过 `config set` 为第二个实例注册独立的浏览器 profile（`cdpPort` 和 `color` 必须以 JSON 对象形式一次写入，分开设置会报 `Config validation failed`）：

```bash
openclaw --profile demo config set browser.profiles.demo '{"cdpPort": 19800, "color": "#0066CC"}'
openclaw --profile demo config set browser.defaultProfile "demo"
openclaw --profile demo gateway restart
openclaw --profile demo browser start
```

验证 `browser status` 中 `profile` 字段显示 `demo` 即成功。

---

## 9. 验证与日常运维

### 查看实例状态

```bash
openclaw status
openclaw --profile demo status
```

### 查看日志

```bash
# macOS 日志路径
tail -f ~/.openclaw-demo/logs/gateway.log
```

### 重启 Gateway

```bash
openclaw --profile demo gateway restart
```

### 查看所有已安装插件

```bash
openclaw --profile demo plugins list
```

### 打开 Web 控制台

```bash
openclaw --profile demo dashboard --no-open
# 输出 URL，在浏览器中打开
```

### 查看与启动 Agent 浏览器

```bash
openclaw --profile demo browser status
openclaw --profile demo browser start
```

---

## 10. 常见问题

### Q: 插件每次更新后需要重新创建符号链接吗？

**A**: 是的。每次执行 `openclaw plugins install @tencent-weixin/openclaw-weixin@latest` 都会重新创建插件目录，需重新建立符号链接：

```bash
cd ~/.openclaw-demo/extensions/openclaw-weixin/node_modules
ln -s $(npm root -g)/openclaw openclaw
openclaw --profile demo gateway restart
```

### Q: 两个实例的浏览器无法同时运行，报 `Failed to start Chrome CDP`？

**A**: 两个实例默认使用同名浏览器 profile（`openclaw`），对应同一个 Chrome 用户数据目录，Chrome 不允许被两个进程同时打开。为第二个实例注册独立的浏览器 profile 即可解决：

```bash
openclaw --profile demo config set browser.profiles.demo '{"cdpPort": 19800, "color": "#0066CC"}'
openclaw --profile demo config set browser.defaultProfile "demo"
openclaw --profile demo gateway restart
```

### Q: 多个 Gateway 可以共享同一个微信账号吗？

**A**: 不建议。每个实例应绑定独立的微信账号，共享账号可能导致消息路由混乱。

### Q: 如何彻底卸载某个 Gateway 实例？

```bash
# 停止并卸载服务
openclaw --profile demo gateway uninstall

# 删除数据目录（谨慎操作）
rm -rf ~/.openclaw-demo
```

### Q: `plugins.allow is empty` 警告是什么意思？

这是提示未配置插件白名单，已安装的插件会自动加载。如需显式控制，执行：

```bash
openclaw --profile demo config set plugins.allow '["openclaw-weixin"]'
```

---

## 完整命令速查

```bash
# === 初始化 demo 实例 ===
openclaw --profile demo onboard
# 注意：Workspace directory 需手动输入 ~/.openclaw-demo/workspace

# === 安装微信插件 ===
openclaw --profile demo plugins install @tencent-weixin/openclaw-weixin@latest

# === 修复插件依赖 ===
cd ~/.openclaw-demo/extensions/openclaw-weixin/node_modules
ln -s $(npm root -g)/openclaw openclaw

# === 重启 Gateway ===
openclaw --profile demo gateway restart

# === 微信扫码登录 ===
openclaw --profile demo channels login --channel openclaw-weixin
```
