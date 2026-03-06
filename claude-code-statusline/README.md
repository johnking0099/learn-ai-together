# Claude Code Status Line 自定义教程

Claude Code 支持在终端底部显示一行自定义状态栏（status line），实时展示模型、token 用量、费用、耗时等关键信息。本教程教你从零配置一个功能完整的 status line。

## 最终效果

配置完成后，你会在 Claude Code 底部看到类似这样的状态栏：

```
[Opus 4.6] 📁 my-project | ▓▓▓░░░░░░░░░░░░ 20%  39.8k/200.0k | 👤 ↑125.3k 🤖 ↓42.1k(+2.5k) ✏️ +36/-12 | 💰 $0.85 ⏳ 1m23s ⏱️ 3m45s
```

各区域含义：

| 区域 | 含义 |
|------|------|
| `[Opus 4.6]` | 当前使用的模型 |
| `📁 my-project` | 当前工作目录名 |
| `▓▓▓░░░░░░░░░░░░ 20%` | 上下文窗口使用进度条 |
| `39.8k/200.0k` | 已用 token / 上下文窗口总大小 |
| `👤 ↑125.3k` | 本次会话累计输入 token 数 |
| `🤖 ↓42.1k(+2.5k)` | 累计输出 token 数（括号内为最近一次） |
| `✏️ +36/-12` | 代码变更行数（新增/删除） |
| `💰 $0.85` | 本次会话累计费用 |
| `⏳ 1m23s` | API 调用累计耗时 |
| `⏱️ 3m45s` | 会话总耗时（含你的思考时间） |

## 前置条件

- Claude Code CLI 已安装（`npm install -g @anthropic-ai/claude-code`）
- 系统已安装 `jq`（用于解析 JSON）
  - macOS: `brew install jq`
  - Ubuntu/Debian: `sudo apt install jq`

## 配置步骤

### 第 1 步：创建 status line 脚本

创建文件 `~/.claude/statusline.sh`：

```bash
#!/bin/bash
# Claude Code Status Line Script
# 从 stdin 读取 Claude Code 传入的 JSON 数据，解析后输出一行状态文本

# 读取 stdin 中的 JSON
input=$(cat)

# ── 基础信息 ──────────────────────────────────────────────
MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
CTX_SIZE=$(echo "$input" | jq -r '.context_window.context_window_size // 0')

# ── 费用与耗时 ────────────────────────────────────────────
COST=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')
DURATION_MS=$(echo "$input" | jq -r '.cost.total_duration_ms // 0')
API_DURATION_MS=$(echo "$input" | jq -r '.cost.total_api_duration_ms // 0')
LINES_ADDED=$(echo "$input" | jq -r '.cost.total_lines_added // 0')
LINES_REMOVED=$(echo "$input" | jq -r '.cost.total_lines_removed // 0')
COST_FMT=$(printf '$%.2f' "$COST")

# ── 当前请求 token 用量 ──────────────────────────────────
LAST_IN=$(echo "$input" | jq -r '.context_window.current_usage.input_tokens // 0')
LAST_OUT=$(echo "$input" | jq -r '.context_window.current_usage.output_tokens // 0')
CACHE_READ=$(echo "$input" | jq -r '.context_window.current_usage.cache_read_input_tokens // 0')
CACHE_WRITE=$(echo "$input" | jq -r '.context_window.current_usage.cache_creation_input_tokens // 0')
USED_TOKENS=$((LAST_IN + CACHE_READ + CACHE_WRITE))

# ── 会话累计 token ────────────────────────────────────────
TOTAL_IN=$(echo "$input" | jq -r '.context_window.total_input_tokens // 0')
TOTAL_OUT=$(echo "$input" | jq -r '.context_window.total_output_tokens // 0')

# ── 辅助函数：毫秒 → 可读时间 ────────────────────────────
time_fmt() {
  local ms=$1
  local total_sec=$((ms / 1000))
  local h=$((total_sec / 3600))
  local m=$(((total_sec % 3600) / 60))
  local s=$((total_sec % 60))
  local result=""
  [ "$h" -gt 0 ] && result+="${h}h"
  [ "$m" -gt 0 ] && result+="${m}m"
  result+="${s}s"
  echo "$result"
}

# ── 辅助函数：数字 → 可读格式（如 200000 → 200.0k）──────
fmt() {
  local n=$1
  if [ "$n" -ge 1000000 ]; then
    printf "%.1fM" "$(echo "$n / 1000000" | bc -l)"
  elif [ "$n" -ge 1000 ]; then
    printf "%.1fk" "$(echo "$n / 1000" | bc -l)"
  else
    echo "$n"
  fi
}

# ── 进度条（15 格宽）─────────────────────────────────────
BAR_WIDTH=15
FILLED=$((PCT * BAR_WIDTH / 100))
EMPTY=$((BAR_WIDTH - FILLED))
BAR=""
[ "$FILLED" -gt 0 ] && BAR=$(printf '%0.s▓' $(seq 1 $FILLED))
[ "$EMPTY" -gt 0 ] && BAR+=$(printf '%0.s░' $(seq 1 $EMPTY))

# ── 组装输出 ──────────────────────────────────────────────
PART1="[$MODEL] 📁 ${DIR##*/}"
PART2="${BAR} ${PCT}%  $(fmt $USED_TOKENS)/$(fmt $CTX_SIZE)"
PART3="👤 ↑$(fmt $TOTAL_IN) 🤖 ↓$(fmt $TOTAL_OUT)(+$(fmt $LAST_OUT)) ✏️ +${LINES_ADDED}/-${LINES_REMOVED}"
PART4="💰 ${COST_FMT} ⏳ $(time_fmt $API_DURATION_MS) ⏱️ $(time_fmt $DURATION_MS)"
STATUS="${PART1} | ${PART2} | ${PART3} | ${PART4}"

echo "$STATUS"
```

设置可执行权限：

```bash
chmod +x ~/.claude/statusline.sh
```

### 第 2 步：配置 Claude Code 使用该脚本

编辑 `~/.claude/settings.json`，添加 `statusLine` 配置项：

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

> **注意**：如果你的 `settings.json` 已有其他配置，只需把 `statusLine` 这一段合并进去即可。

### 第 3 步：重启 Claude Code

关闭当前 Claude Code 会话，重新打开即可看到状态栏。

## 工作原理

1. Claude Code 每次状态变化时，会将一个 JSON 对象通过 **stdin** 传给你指定的脚本
2. 脚本解析 JSON，提取需要的字段，格式化后输出一行文本到 **stdout**
3. Claude Code 将这行文本渲染在终端底部

### Claude Code 传入的 JSON 结构

以下是 `statusLine` 脚本通过 stdin 收到的 JSON 数据中可用的字段：

```jsonc
{
  "model": {
    "display_name": "Opus 4.6",     // 模型显示名
    "model_id": "claude-opus-4-6"   // 模型 ID
  },
  "workspace": {
    "current_dir": "/Users/you/project"  // 当前工作目录
  },
  "context_window": {
    "used_percentage": 20.5,             // 上下文窗口使用百分比
    "context_window_size": 200000,       // 上下文窗口总大小（token 数）
    "current_usage": {
      "input_tokens": 15000,             // 最近一次请求的输入 token
      "output_tokens": 2500,             // 最近一次请求的输出 token
      "cache_read_input_tokens": 20000,  // 缓存命中的 token
      "cache_creation_input_tokens": 5000 // 新写入缓存的 token
    },
    "total_input_tokens": 125000,        // 会话累计输入 token
    "total_output_tokens": 42000         // 会话累计输出 token
  },
  "cost": {
    "total_cost_usd": 0.85,             // 会话累计费用（美元）
    "total_duration_ms": 225000,         // 会话总耗时（毫秒）
    "total_api_duration_ms": 83000,      // API 调用累计耗时（毫秒）
    "total_lines_added": 36,             // 代码新增行数
    "total_lines_removed": 12            // 代码删除行数
  }
}
```

## 自定义调整

### 只显示部分信息

如果你觉得信息太多，可以只保留需要的部分。例如只显示模型 + 进度条 + 费用：

```bash
STATUS="[$MODEL] ${BAR} ${PCT}% | ${COST_FMT}"
echo "$STATUS"
```

### 调整进度条宽度

修改 `BAR_WIDTH` 的值：

```bash
BAR_WIDTH=20   # 更宽的进度条
BAR_WIDTH=10   # 更窄的进度条
```

### 更换进度条样式

```bash
# 样式 1：实心/空心方块
BAR=$(printf '%0.s█' $(seq 1 $FILLED))$(printf '%0.s░' $(seq 1 $EMPTY))

# 样式 2：等号和横线
BAR=$(printf '%0.s=' $(seq 1 $FILLED))$(printf '%0.s-' $(seq 1 $EMPTY))

# 样式 3：带括号
BAR="[$(printf '%0.s#' $(seq 1 $FILLED))$(printf '%0.s.' $(seq 1 $EMPTY))]"
```

### 上下文窗口用量过高时变色提醒

如果你的终端支持 ANSI 颜色，可以在 `PCT` 较高时加颜色提醒：

```bash
RED='\033[0;31m'
YELLOW='\033[0;33m'
GREEN='\033[0;32m'
NC='\033[0m'  # 恢复默认

if [ "$PCT" -ge 80 ]; then
  COLOR=$RED
elif [ "$PCT" -ge 50 ]; then
  COLOR=$YELLOW
else
  COLOR=$GREEN
fi

PART2="${COLOR}${BAR} ${PCT}%${NC}  $(fmt $USED_TOKENS)/$(fmt $CTX_SIZE)"
```

## 调试技巧

如果 status line 没有正常显示，可以手动测试脚本：

```bash
# 用一个示例 JSON 测试你的脚本
echo '{"model":{"display_name":"Opus 4.6"},"workspace":{"current_dir":"/tmp/test"},"context_window":{"used_percentage":25,"context_window_size":200000,"current_usage":{"input_tokens":10000,"output_tokens":2000,"cache_read_input_tokens":30000,"cache_creation_input_tokens":5000},"total_input_tokens":80000,"total_output_tokens":20000},"cost":{"total_cost_usd":0.42,"total_duration_ms":120000,"total_api_duration_ms":45000,"total_lines_added":15,"total_lines_removed":3}}' | ~/.claude/statusline.sh
```

常见问题：
- **没有输出**：检查 `jq` 是否已安装（`which jq`）
- **权限错误**：确认脚本有可执行权限（`chmod +x ~/.claude/statusline.sh`）
- **显示为空**：检查 `settings.json` 格式是否正确（用 `jq . ~/.claude/settings.json` 验证）
