> 返回 [README 总览](./README.md)

# 01 - 环境搭建

本篇使用 **uv** 作为 Python 安装和项目管理工具。uv 是一个用 Rust 编写的现代 Python 工具链，速度极快，能替代 pip、virtualenv、pyenv 等多个工具。

本篇仅覆盖 **macOS** 和 **Linux** 环境。

---

## 安装 uv

在终端执行：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

安装完成后重新打开终端，验证：

```bash
uv --version
```

---

## 用 uv 安装 Python

```bash
uv python install 3.12
```

推荐使用 Python **3.10+**，原因：
- 3.10 引入了 `match` 语句和更友好的错误提示
- 3.10+ 支持 `X | Y` 类型标注语法，代码更简洁
- 主流框架（FastAPI、Pydantic 等）都要求 3.8+，但 3.12 性能更好

验证安装：

```bash
uv python list     # 查看已安装的 Python 版本
python --version   # 查看默认 Python 版本
```

---

## uv 项目管理

### 创建新项目

```bash
uv init my-project    # 创建项目目录和基础文件
cd my-project
```

这会生成一个包含 `pyproject.toml` 和 `hello.py` 的目录。

### 常用命令

```bash
uv add requests       # 添加依赖（类似 npm install）
uv add ruff --dev     # 添加开发依赖
uv remove requests    # 移除依赖
uv sync               # 同步所有依赖（类似 npm install）
uv run python app.py  # 在项目环境中运行脚本
```

`uv run` 会自动使用项目的虚拟环境，确保依赖隔离。

---

## pyproject.toml

这是现代 Python 项目的标准配置文件，作用类似 Node.js 的 `package.json`：

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.31.0",
    "fastapi>=0.110.0",
]

[dependency-groups]
dev = [
    "ruff>=0.4.0",
    "pytest>=8.0.0",
]
```

你不需要手动编辑这个文件——`uv add` / `uv remove` 会自动维护它。

---

## 虚拟环境

uv 会自动在项目目录下创建 `.venv` 目录，无需手动操作。虚拟环境的作用是让每个项目有独立的依赖，互不干扰。

如果你看到别人的教程提到 `python -m venv` 或 `virtualenv`，知道它们是手动创建虚拟环境的传统方式即可。用 uv 时不需要关心这些。

---

## requirements.txt

很多现有项目仍在使用 `requirements.txt` 管理依赖：

```text
requests==2.31.0
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
```

遇到这类项目时，用 uv 安装依赖：

```bash
uv pip install -r requirements.txt
```

---

## 编辑器推荐

推荐使用 **VS Code** + **Python 扩展**（Microsoft 官方）：

- 语法高亮、自动补全
- 类型检查和错误提示
- 集成终端、调试器
- 丰富的插件生态

---

## 两种运行方式

### REPL 交互式

```bash
uv run python
```

进入交互环境后可以逐行执行代码，适合快速验证想法：

```python
>>> 1 + 1
2
>>> "hello".upper()
'HELLO'
>>> exit()
```

### 脚本执行

创建一个 `app.py` 文件：

```python
name = "World"
print(f"Hello, {name}!")
```

运行：

```bash
uv run python app.py
```

---

## `if __name__ == "__main__":` 惯用法

在真实项目中你会频繁看到这个写法：

```python
def main():
    print("程序开始运行")

if __name__ == "__main__":
    main()
```

**它的含义**：只有当这个文件被直接运行时，`main()` 才会执行。如果这个文件被其他文件 `import`，`main()` 不会自动执行。

这是 Python 项目中最常见的入口点写法，几乎每个可运行的脚本都会用到。

---

## 典型项目目录结构

一个真实的 Python 项目通常长这样：

```
my-project/
├── pyproject.toml          # 项目配置和依赖
├── uv.lock                 # 依赖锁定文件（自动生成）
├── README.md               # 项目说明
├── src/
│   └── my_project/         # 源代码目录
│       ├── __init__.py     # 包标识文件
│       ├── main.py         # 入口文件
│       ├── models.py       # 数据模型
│       └── utils.py        # 工具函数
├── tests/                  # 测试目录
│   └── test_main.py
└── .venv/                  # 虚拟环境（自动生成，不提交到 Git）
```

你现在不需要理解每个文件的作用——在后续章节中会逐步接触到它们。

---

> 下一篇：[02 - 变量与基本数据类型](./02-variables-and-types.md)
