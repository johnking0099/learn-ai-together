> 返回 [README 总览](./README.md)

# 06 - 模块与包管理

当代码量增长，你需要把代码拆分到多个文件。Python 的 **模块**（module）和 **包**（package）机制就是做这件事的。

---

## import 语句

### 基本导入

```python
import os                      # 导入整个模块
print(os.getcwd())             # 通过模块名访问

import json
data = json.loads('{"a": 1}')
```

### 从模块导入特定内容

```python
from os import path, getcwd    # 只导入需要的部分
print(getcwd())                # 直接使用，不需要前缀

from datetime import datetime
now = datetime.now()
```

### 别名

```python
import numpy as np             # 常见惯例：numpy 简写为 np
import pandas as pd            # pandas 简写为 pd
from datetime import datetime as dt
```

### 不推荐：import *

```python
from os import *               # 导入模块的所有内容 — 不要这样做！
```

`import *` 会污染当前命名空间，让人搞不清某个函数来自哪里。在真实项目中应避免使用。

---

## 模块搜索路径

当你写 `import something` 时，Python 按以下顺序查找：

1. 当前目录
2. 已安装的第三方包（`site-packages`）
3. 标准库
4. `sys.path` 中的其他路径

```python
import sys
print(sys.path)    # 查看所有搜索路径
```

大多数时候你不需要关心搜索路径。但如果遇到 `ModuleNotFoundError`，检查这个列表有时能帮你定位问题。

---

## 包的结构

**包**是包含 `__init__.py` 文件的目录。一个真实项目的目录结构：

```
my_project/
├── __init__.py            # 标识这是一个 Python 包（可以为空文件）
├── main.py
├── models/
│   ├── __init__.py
│   ├── user.py
│   └── order.py
├── services/
│   ├── __init__.py
│   ├── auth.py
│   └── payment.py
└── utils/
    ├── __init__.py
    └── helpers.py
```

导入方式：

```python
from my_project.models.user import User
from my_project.services.auth import authenticate
from my_project.utils.helpers import format_date
```

### `__init__.py` 的作用

- 最基本：空文件，仅标识目录为 Python 包
- 进阶：可以在其中定义包的公开接口

```python
# my_project/models/__init__.py
from .user import User
from .order import Order

# 这样外部可以直接写：
from my_project.models import User, Order
# 而不需要：
from my_project.models.user import User
```

---

## 相对导入 vs 绝对导入

```python
# 绝对导入（推荐）— 从项目根目录开始
from my_project.models.user import User

# 相对导入 — 用 . 表示当前包
from .user import User         # 当前目录的 user.py
from ..utils import helpers    # 上级目录的 utils 包
```

**推荐使用绝对导入**：路径明确，不容易出错，IDE 支持更好。

相对导入（`.` 和 `..`）在某些框架内部代码中会看到，认识即可。

---

## import 分组惯例

Python 社区约定将 import 语句分为三组，用空行分隔：

```python
# 1. 标准库
import os
import json
from datetime import datetime
from pathlib import Path

# 2. 第三方库
import requests
from fastapi import FastAPI
from pydantic import BaseModel

# 3. 本地模块
from my_project.models import User
from my_project.services.auth import authenticate
```

这个分组方式让人一眼就能区分依赖来源。代码格式化工具（如 `ruff`、`isort`）可以自动帮你排列 import 顺序。

---

> 下一篇：[07 - 类与面向对象](./07-classes-and-objects.md)
