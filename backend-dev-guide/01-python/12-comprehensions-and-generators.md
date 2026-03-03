> 返回 [README 总览](./README.md)

# 12 - 推导式与生成器

推导式（comprehension）是 Python 最具特色的语法之一，能把多行循环压缩成简洁的一行。

---

## 列表推导式（重点）

### 基本语法

```python
# 传统写法
squares = []
for x in range(10):
    squares.append(x ** 2)

# 列表推导式
squares = [x ** 2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

模式：`[表达式 for 变量 in 可迭代对象]`

### 带条件过滤

```python
# 只保留偶数
evens = [x for x in range(20) if x % 2 == 0]
# [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# 过滤 + 转换
names = ["Alice", "", "Bob", "", "Charlie"]
valid_names = [name.upper() for name in names if name]
# ["ALICE", "BOB", "CHARLIE"]
```

模式：`[表达式 for 变量 in 可迭代对象 if 条件]`

### 嵌套推导式

```python
# 二维列表展平
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**建议**：嵌套推导式可读性差，超过两层嵌套时改用普通循环。代码的可读性比简洁更重要。

---

## 字典推导式

```python
# 基本用法
names = ["Alice", "Bob", "Charlie"]
name_lengths = {name: len(name) for name in names}
# {"Alice": 5, "Bob": 3, "Charlie": 7}

# 反转字典
original = {"a": 1, "b": 2, "c": 3}
reversed_dict = {v: k for k, v in original.items()}
# {1: "a", 2: "b", 3: "c"}

# 带过滤
scores = {"Alice": 95, "Bob": 60, "Charlie": 85}
excellent = {name: score for name, score in scores.items() if score >= 80}
# {"Alice": 95, "Charlie": 85}
```

---

## 集合推导式

```python
words = ["hello", "world", "hello", "python"]
unique_lengths = {len(word) for word in words}
# {5, 6}
```

---

## 生成器表达式

把列表推导式的方括号 `[]` 换成圆括号 `()`，就变成了**生成器表达式**：

```python
# 列表推导式：立即创建整个列表，占用内存
squares_list = [x ** 2 for x in range(1000000)]

# 生成器表达式：惰性求值，用一个算一个
squares_gen = (x ** 2 for x in range(1000000))
```

**惰性求值**：生成器不会一次性计算所有结果，而是每次调用时计算一个。处理大量数据时，这能节省大量内存。

```python
# 常见用法：配合 sum()、max() 等函数
total = sum(x ** 2 for x in range(1000))   # 不需要额外的括号
largest = max(len(name) for name in names)
```

---

## yield 与生成器函数

`yield` 把普通函数变成**生成器函数**——函数不会一次性执行完，而是每次 `yield` 时暂停，返回一个值：

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for num in countdown(5):
    print(num)       # 5, 4, 3, 2, 1
```

### 生成器 vs 普通函数

```python
# 普通函数：一次性返回所有结果
def get_all_lines(filename):
    with open(filename) as f:
        return f.readlines()    # 全部读入内存

# 生成器函数：逐行产出，内存友好
def read_lines(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()  # 每次只处理一行
```

---

## 真实场景

### 数据过滤 one-liner

```python
# 从 API 响应中提取活跃用户的邮箱
active_emails = [
    user["email"]
    for user in response["users"]
    if user["is_active"]
]
```

### FastAPI 依赖注入中的 yield

在 FastAPI 中，`yield` 用于管理资源的生命周期（衔接 [02 - Web 框架](../02-web-framework/)）：

```python
def get_db():
    db = SessionLocal()
    try:
        yield db           # 请求处理期间使用 db
    finally:
        db.close()          # 请求结束后自动关闭

@app.get("/users")
def list_users(db=Depends(get_db)):
    return db.query(User).all()
```

`yield` 前面的代码在请求开始时执行，`yield` 后面的代码在请求结束时执行——这是一种巧妙的资源管理模式。

---

> 下一篇：[13 - 异步编程基础](./13-async-basics.md)
