> 返回 [README 总览](./README.md)

# 03 - 核心数据结构

Python 有四种内置容器类型：**list**（列表）、**tuple**（元组）、**dict**（字典）、**set**（集合）。掌握它们是写 Python 代码的基础。

---

## list（列表）

有序、可变的序列，用方括号创建：

```python
fruits = ["apple", "banana", "cherry"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", True, 3.14]    # 可以混合类型（但不推荐）
```

### 索引与切片

```python
fruits = ["apple", "banana", "cherry", "date"]

fruits[0]       # "apple"       第一个元素（从 0 开始）
fruits[-1]      # "date"        最后一个元素
fruits[1:3]     # ["banana", "cherry"]  切片：索引 1 到 2（不含 3）
fruits[:2]      # ["apple", "banana"]   从开头到索引 1
fruits[2:]      # ["cherry", "date"]    从索引 2 到结尾
```

**负索引**是 Python 的特色：`-1` 是最后一个，`-2` 是倒数第二个，以此类推。

### 常用方法

```python
fruits = ["apple", "banana"]

fruits.append("cherry")        # 末尾添加：["apple", "banana", "cherry"]
fruits.extend(["date", "fig"]) # 扩展多个元素
fruits.insert(1, "avocado")    # 在索引 1 处插入
fruits.pop()                   # 移除并返回最后一个元素
fruits.pop(0)                  # 移除并返回索引 0 的元素
fruits.sort()                  # 原地排序
fruits.reverse()               # 原地反转

len(fruits)                    # 列表长度
"apple" in fruits              # True，检查元素是否存在
```

---

## tuple（元组）

有序、**不可变**的序列，用圆括号创建：

```python
point = (3, 4)
rgb = (255, 128, 0)
single = (42,)        # 注意：单元素 tuple 需要逗号
```

### 解包（Unpacking）

```python
x, y = (3, 4)         # x=3, y=4
name, age, city = ("Alice", 25, "Beijing")
```

### 函数多返回值

函数返回多个值时，实际上返回的是 tuple：

```python
def get_user():
    return "Alice", 25    # 返回 tuple

name, age = get_user()    # 解包接收
```

tuple 的核心用途：需要不可变序列的场景（如字典的键、函数返回多个值）。

---

## dict（字典）

**键值对**的集合，Python 中最常用的数据结构之一：

```python
user = {
    "name": "Alice",
    "age": 25,
    "email": "alice@example.com",
}
```

### 访问与修改

```python
user["name"]               # "Alice"    直接访问（键不存在会报错）
user.get("name")           # "Alice"    安全访问
user.get("phone", "N/A")   # "N/A"      键不存在时返回默认值

user["age"] = 26           # 修改
user["phone"] = "123456"   # 新增键值对
del user["email"]          # 删除
```

**重要**：始终优先用 `.get()` 安全访问，避免 `KeyError`。

### 遍历字典

```python
user = {"name": "Alice", "age": 25, "city": "Beijing"}

# 遍历键值对（最常用）
for key, value in user.items():
    print(f"{key}: {value}")

# 遍历键
for key in user.keys():
    print(key)

# 遍历值
for value in user.values():
    print(value)
```

### 嵌套字典

真实项目中，字典经常嵌套使用（想象 JSON 数据）：

```python
response = {
    "status": 200,
    "data": {
        "users": [
            {"name": "Alice", "age": 25},
            {"name": "Bob", "age": 30},
        ]
    }
}

# 逐层访问
first_user = response["data"]["users"][0]["name"]  # "Alice"
```

---

## set（集合）

无序、不重复的元素集合：

```python
tags = {"python", "backend", "ai"}
numbers = {1, 2, 3, 2, 1}     # 自动去重：{1, 2, 3}
```

### 常用操作

```python
tags.add("docker")             # 添加元素
tags.discard("ai")             # 移除元素（不存在不报错）
"python" in tags               # True，检查是否存在

# 集合运算
a = {1, 2, 3}
b = {2, 3, 4}
a | b        # {1, 2, 3, 4}   并集
a & b        # {2, 3}         交集
a - b        # {1}            差集
```

set 最常见的用途是**去重**：

```python
names = ["Alice", "Bob", "Alice", "Charlie", "Bob"]
unique_names = list(set(names))   # ["Alice", "Bob", "Charlie"]（顺序不保证）
```

---

## 可变 vs 不可变

| 类型 | 可变？ | 说明 |
|------|-------|------|
| `list` | 可变 | 可以增删改元素 |
| `dict` | 可变 | 可以增删改键值对 |
| `set` | 可变 | 可以增删元素 |
| `tuple` | 不可变 | 创建后不能修改 |
| `str` | 不可变 | 字符串方法返回新字符串 |
| `int`/`float` | 不可变 | 赋值是创建新对象 |

### 经典陷阱：函数默认参数不要用可变对象

```python
# 错误写法！
def add_item(item, items=[]):
    items.append(item)
    return items

add_item("a")   # ["a"]
add_item("b")   # ["a", "b"]  ← 不是预期的 ["b"]！

# 正确写法
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

默认参数在函数定义时创建一次，之后每次调用共享同一个对象。这是 Python 最经典的"坑"之一。

---

> 下一篇：[04 - 控制流](./04-control-flow.md)
