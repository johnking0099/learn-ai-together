> 返回 [README 总览](./README.md)

# 11 - 类型标注

Python 是动态类型语言，但从 3.5 开始引入了**类型标注**（Type Hints）。类型标注不影响运行，但能极大提升代码可读性和 IDE 体验。

---

## 为什么需要类型标注

```python
# 没有类型标注：看不出参数和返回值是什么类型
def process(data, threshold):
    ...

# 有类型标注：一目了然
def process(data: list[dict], threshold: float) -> list[str]:
    ...
```

类型标注的好处：

1. **可读性**：不用猜参数类型，读代码更快
2. **IDE 支持**：自动补全、错误提示、重构辅助
3. **框架依赖**：Pydantic、FastAPI 等框架**强依赖**类型标注来实现功能

---

## 基本标注语法

### 变量标注

```python
name: str = "Alice"
age: int = 25
ratio: float = 0.95
is_active: bool = True
```

### 函数标注

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

def add(a: int, b: int) -> int:
    return a + b

def log(message: str) -> None:    # 无返回值用 None
    print(message)
```

---

## 常用类型

### 容器类型

```python
# Python 3.9+ 可以直接用内置类型
names: list[str] = ["Alice", "Bob"]
scores: dict[str, int] = {"Alice": 95, "Bob": 87}
point: tuple[float, float] = (3.14, 2.71)
tags: set[str] = {"python", "ai"}

# 嵌套类型
users: list[dict[str, str]] = [
    {"name": "Alice", "email": "alice@example.com"},
]
```

### Optional 和 None 联合类型

表示"值可能是某个类型，也可能是 None"：

```python
# Python 3.10+ 推荐写法
def find_user(user_id: int) -> dict | None:
    ...

# 兼容旧版本的写法
from typing import Optional
def find_user(user_id: int) -> Optional[dict]:
    ...
```

`X | None` 和 `Optional[X]` 含义完全相同。新项目推荐用 `X | None`。

### 联合类型

```python
# 接受多种类型
def process(value: int | str) -> str:
    return str(value)
```

---

## typing 模块常用类型

```python
from typing import Any, Callable, TypeVar

# Any：任意类型（少用，降低了类型标注的价值）
def log(data: Any) -> None:
    print(data)

# Callable：函数类型
# Callable[[参数类型...], 返回类型]
def apply(func: Callable[[int], str], value: int) -> str:
    return func(value)
```

### TypeVar（简述）

用于定义泛型，你在框架源码中可能会看到：

```python
from typing import TypeVar

T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]

# first(["a", "b"]) 返回 str
# first([1, 2, 3]) 返回 int
```

看到 `TypeVar` 时，理解为"这是一个类型占位符"即可，不需要深入。

---

## Pydantic BaseModel

Pydantic 是一个数据验证库，**FastAPI 的核心依赖**。它利用类型标注自动做数据验证和转换：

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str | None = None

# 自动验证
user = User(name="Alice", age=25)
print(user.name)           # "Alice"
print(user.model_dump())   # {"name": "Alice", "age": 25, "email": None}

# 类型不对会报错
User(name="Alice", age="abc")  # ValidationError
```

对比 `@dataclass`（[第 07 章](./07-classes-and-objects.md)）：

| 特性 | dataclass | Pydantic BaseModel |
|------|-----------|-------------------|
| 数据验证 | 无 | 自动验证类型和约束 |
| JSON 序列化 | 需手动 | 内置 `model_dump()` / `model_dump_json()` |
| 使用场景 | 内部数据结构 | API 输入输出、配置文件 |

在 [02 - Web 框架](../02-web-framework/) 章节中会更深入地使用 Pydantic。

---

## 阅读技巧：如何解读复杂类型签名

遇到看不懂的类型签名时，从内向外拆解：

```python
def process_batch(
    items: list[dict[str, int | float]],
    callback: Callable[[str, float], None] | None = None,
) -> dict[str, list[float]]:
    ...
```

拆解过程：

1. `items`: 一个 list，元素是 dict，键是 str，值是 int 或 float
2. `callback`: 一个函数（接收 str 和 float，无返回值），或者 None，默认 None
3. 返回值：一个 dict，键是 str，值是 float 列表

复杂的类型签名在框架代码中很常见，不需要一次看懂——先理解参数名和函数名，再回头看类型。

---

> 下一篇：[12 - 推导式与生成器](./12-comprehensions-and-generators.md)
