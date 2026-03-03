> 返回 [README 总览](./README.md)

# 10 - 装饰器与上下文管理器

这是"能读懂真实项目代码"的关键章节。装饰器和上下文管理器在 Python 框架中无处不在。

---

## 装饰器是什么

装饰器的本质是"**函数包函数**"——接收一个函数，返回一个增强版的函数。

### 直觉理解

```python
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)           # 调用原函数
        print(f"{func.__name__} 耗时 {time.time() - start:.2f}s")
        return result
    return wrapper
```

### @decorator 语法糖

`@decorator` 只是一种简写：

```python
# 这两种写法完全等价

# 写法一：@ 语法糖
@timer
def slow_function():
    time.sleep(1)

# 写法二：手动包装
def slow_function():
    time.sleep(1)
slow_function = timer(slow_function)
```

看到 `@xxx` 时，在脑中展开为 `func = xxx(func)` 就能理解。

---

## 常见内置装饰器

### @property

把方法变成属性访问：

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2

c = Circle(5)
print(c.area)       # 78.53975    像属性一样访问，不需要加 ()
```

### @staticmethod 和 @classmethod

已在[第 07 章](./07-classes-and-objects.md)介绍，这里不重复。

---

## 带参数的装饰器

有时你会看到三层嵌套的装饰器——这是因为最外层用于接收装饰器自己的参数：

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def say_hello():
    print("Hello!")

say_hello()    # 打印 3 次 "Hello!"
```

**你不需要能写出这种三层嵌套**。只需要知道：当 `@decorator(参数)` 带了参数时，它比普通装饰器多嵌套一层。

---

## 真实代码中的装饰器（重点）

理解装饰器的最大价值在于**读懂框架代码**。以下是你在真实项目中一定会遇到的：

### FastAPI 路由

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"user_id": user_id}

@app.post("/users")
async def create_user(user: UserCreate):
    return {"message": "创建成功"}
```

`@app.get("/users/{user_id}")` 的含义：把这个函数注册为处理 `GET /users/{user_id}` 请求的处理器。

### pytest 测试

```python
import pytest

@pytest.fixture
def database():
    db = create_test_db()
    yield db
    db.cleanup()

@pytest.mark.parametrize("input,expected", [(1, 1), (2, 4), (3, 9)])
def test_square(input, expected):
    assert input ** 2 == expected
```

### click 命令行工具

```python
import click

@click.command()
@click.option("--name", prompt="你的名字", help="用户名")
@click.option("--count", default=1, help="重复次数")
def hello(name, count):
    for _ in range(count):
        click.echo(f"Hello, {name}!")
```

这些框架的共同模式：**用装饰器声明"这个函数是什么角色"**，框架负责在合适的时候调用它。

---

## 上下文管理器

### with 语句

你已经在[第 09 章](./09-file-and-io.md)用过 `with open(...)`——`with` 语句保证资源会被正确清理：

```python
# 文件会在 with 块结束时自动关闭
with open("data.txt") as f:
    content = f.read()

# 即使发生异常，文件也会被关闭
```

### 原理：__enter__ 和 __exit__

`with` 语句调用对象的两个魔术方法：

```python
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_to_db()
        return self.conn

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()        # 无论是否有异常，都会关闭连接

# 使用
with DatabaseConnection() as conn:
    conn.execute("SELECT * FROM users")
# 离开 with 块后，连接自动关闭
```

### 真实场景

```python
# 数据库事务
with db.begin() as transaction:
    transaction.execute(...)
    transaction.execute(...)
# 正常结束 → 自动 commit；发生异常 → 自动 rollback

# 临时文件
import tempfile
with tempfile.NamedTemporaryFile() as tmp:
    tmp.write(b"临时数据")
# 离开 with 块后，临时文件自动删除

# 线程锁
import threading
lock = threading.Lock()
with lock:
    # 线程安全的操作
    shared_data.update(...)
# 离开 with 块后，锁自动释放
```

上下文管理器的核心价值：**确保资源一定会被释放**，即使中途发生异常。

---

> 下一篇：[11 - 类型标注](./11-type-hints.md)
