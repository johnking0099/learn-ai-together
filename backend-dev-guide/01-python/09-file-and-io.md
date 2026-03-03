> 返回 [README 总览](./README.md)

# 09 - 文件操作与常用标准库

文件读写和标准库是日常开发的基本功。本篇覆盖最常用的部分。

---

## 文件读写

### open() 基本用法

```python
# 读取文件
f = open("data.txt", "r", encoding="utf-8")
content = f.read()
f.close()          # 必须关闭文件

# 写入文件
f = open("output.txt", "w", encoding="utf-8")
f.write("Hello, World!\n")
f.close()
```

模式参数：`"r"` 读取、`"w"` 写入（覆盖）、`"a"` 追加、`"rb"` 读取二进制。

### with 语句（推荐写法）

```python
# 自动关闭文件，不需要手动 close()
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()

# 离开 with 块后，文件自动关闭
```

**始终用 `with open(...)` 而不是手动 `open()` + `close()`**。这是 Python 的惯用法，第 10 章会详细解释 `with` 的原理。

### 常用读取方式

```python
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()          # 读取全部内容为字符串
    lines = f.readlines()       # 读取所有行为列表

# 逐行读取（内存友好，适合大文件）
with open("data.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())     # strip() 去除行尾换行符
```

**encoding 参数**：始终显式指定 `encoding="utf-8"`，避免不同系统默认编码不同导致的乱码问题。

---

## pathlib（推荐的路径处理方式）

`pathlib` 是 Python 3.4+ 引入的面向对象路径库，比传统的 `os.path` 更直观：

```python
from pathlib import Path

# 创建路径对象
project_dir = Path("/home/user/project")
config_file = project_dir / "config" / "settings.json"    # 用 / 拼接路径

# 常用操作
config_file.exists()         # 文件是否存在
config_file.is_file()        # 是否是文件
config_file.is_dir()         # 是否是目录
config_file.name              # "settings.json"
config_file.stem              # "settings"
config_file.suffix            # ".json"
config_file.parent            # Path("/home/user/project/config")
```

### 快捷读写

```python
# 读取文件内容（一行搞定）
content = Path("data.txt").read_text(encoding="utf-8")

# 写入文件
Path("output.txt").write_text("Hello!", encoding="utf-8")
```

### 遍历目录

```python
# 列出目录下所有 .py 文件
for py_file in Path("src").glob("*.py"):
    print(py_file)

# 递归查找所有 .py 文件
for py_file in Path("src").rglob("*.py"):
    print(py_file)
```

---

## json 模块

JSON 是 Web 开发中最常用的数据格式，`json` 模块用得极其频繁：

```python
import json

# 字符串 ↔ Python 对象
data = json.loads('{"name": "Alice", "age": 25}')   # JSON 字符串 → dict
text = json.dumps(data, ensure_ascii=False)           # dict → JSON 字符串

# 文件 ↔ Python 对象
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)                               # 从文件读取

with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)   # 写入文件（格式化）
```

记忆技巧：
- `loads` / `dumps`：s 代表 **s**tring，处理字符串
- `load` / `dump`：没有 s，处理**文件**
- `ensure_ascii=False`：让中文正常显示而不是转义为 `\uXXXX`

---

## 环境变量

在 AI 后端项目中，API 密钥、数据库地址等敏感信息通常通过环境变量传递：

```python
import os

# 读取环境变量
api_key = os.environ["API_KEY"]           # 不存在会报 KeyError
api_key = os.getenv("API_KEY")            # 不存在返回 None
api_key = os.getenv("API_KEY", "default") # 不存在返回默认值

# 设置环境变量（仅当前进程有效）
os.environ["DEBUG"] = "true"
```

真实项目中常见模式：

```python
import os

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///dev.db")
DEBUG = os.getenv("DEBUG", "false").lower() == "true"
```

---

## 常用标准库速览

### datetime — 日期时间

```python
from datetime import datetime, timedelta

now = datetime.now()
print(now.strftime("%Y-%m-%d %H:%M:%S"))    # "2024-01-15 14:30:00"

# 时间运算
tomorrow = now + timedelta(days=1)
one_hour_later = now + timedelta(hours=1)
```

### collections — 实用容器

```python
from collections import defaultdict, Counter

# defaultdict：访问不存在的键时自动创建默认值
word_count = defaultdict(int)
for word in ["apple", "banana", "apple", "cherry", "apple"]:
    word_count[word] += 1
# {"apple": 3, "banana": 1, "cherry": 1}

# Counter：计数器（更简洁的写法）
counter = Counter(["apple", "banana", "apple", "cherry", "apple"])
counter.most_common(2)    # [("apple", 3), ("banana", 1)]
```

### re — 正则表达式

```python
import re

# 查找匹配
email = "alice@example.com"
if re.match(r"[\w.]+@[\w.]+", email):
    print("是邮箱格式")

# 查找所有匹配
text = "价格：100元，折后：80元"
prices = re.findall(r"\d+", text)    # ["100", "80"]
```

正则表达式是一个独立的话题。入门阶段只需知道 `re.match()`、`re.search()`、`re.findall()` 这几个函数即可。

### logging — 日志

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("服务启动")
logger.warning("连接超时，正在重试")
logger.error("数据库连接失败")
```

比 `print()` 更适合正式项目——日志可以设置级别、输出到文件、格式化时间戳等。

---

> 下一篇：[10 - 装饰器与上下文管理器](./10-decorators-and-context.md)
