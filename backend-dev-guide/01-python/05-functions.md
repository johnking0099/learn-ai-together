> 返回 [README 总览](./README.md)

# 05 - 函数

函数是组织和复用代码的基本单元。

---

## 定义与调用

```python
def greet(name):
    return f"Hello, {name}!"

message = greet("Alice")
print(message)              # "Hello, Alice!"
```

### 无返回值

不写 `return` 或写 `return` 不带值，函数返回 `None`：

```python
def log(message):
    print(f"[LOG] {message}")
    # 隐式返回 None

result = log("启动")     # 打印日志，result 为 None
```

---

## 参数类型

### 位置参数与关键字参数

```python
def create_user(name, age, city="Beijing"):
    return {"name": name, "age": age, "city": city}

# 位置参数：按顺序传
create_user("Alice", 25)

# 关键字参数：按名字传（顺序无所谓）
create_user(age=25, name="Alice")

# 混合使用：位置参数必须在前
create_user("Alice", 25, city="Shanghai")
```

### 默认值参数

```python
def connect(host="localhost", port=8080, timeout=30):
    print(f"连接 {host}:{port}，超时 {timeout}s")

connect()                          # 全部用默认值
connect(port=3000)                 # 只改 port
connect("192.168.1.1", 9090)       # 改 host 和 port
```

---

## *args 和 **kwargs（重点）

这是初学者读代码时**最大的困惑点之一**，但理解了就很简单。

### *args：接收任意数量的位置参数

```python
def add(*args):
    print(type(args))    # <class 'tuple'>
    return sum(args)

add(1, 2)           # 3
add(1, 2, 3, 4, 5)  # 15
```

`*args` 把多余的位置参数收集成一个 **tuple**。

### **kwargs：接收任意数量的关键字参数

```python
def create_config(**kwargs):
    print(type(kwargs))    # <class 'dict'>
    for key, value in kwargs.items():
        print(f"  {key} = {value}")

create_config(host="localhost", port=8080, debug=True)
```

`**kwargs` 把多余的关键字参数收集成一个 **dict**。

### 组合使用

```python
def flexible(required, *args, **kwargs):
    print(f"必选：{required}")
    print(f"额外位置参数：{args}")
    print(f"额外关键字参数：{kwargs}")

flexible("hello", 1, 2, 3, name="Alice", age=25)
# 必选：hello
# 额外位置参数：(1, 2, 3)
# 额外关键字参数：{'name': 'Alice', 'age': 25}
```

**真实场景**：很多框架用 `*args` 和 `**kwargs` 做参数透传，看到时不要慌——它只是"把参数打包/解包"。

---

## Lambda 表达式

匿名的一行函数，常配合 `sorted()` 使用：

```python
# 完整写法
def get_age(user):
    return user["age"]
sorted(users, key=get_age)

# lambda 简写
sorted(users, key=lambda user: user["age"])
```

更多示例：

```python
users = [
    {"name": "Charlie", "age": 35},
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30},
]

# 按年龄排序
sorted(users, key=lambda u: u["age"])

# 按名字排序
sorted(users, key=lambda u: u["name"])

# 按年龄降序
sorted(users, key=lambda u: u["age"], reverse=True)
```

lambda 只用于简单场景。如果逻辑复杂，应该用普通 `def` 定义函数。

---

## 函数作为参数（高阶函数）

Python 中函数是"一等公民"，可以像变量一样传递：

```python
def apply(func, value):
    return func(value)

apply(str.upper, "hello")    # "HELLO"
apply(len, [1, 2, 3])        # 3
```

### map() 和 filter()

```python
numbers = [1, 2, 3, 4, 5]

# map：对每个元素应用函数
doubled = list(map(lambda x: x * 2, numbers))    # [2, 4, 6, 8, 10]

# filter：过滤元素
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]
```

在实际代码中，**列表推导式**（第 12 章）通常比 `map`/`filter` 更常用、更易读。

---

## 返回多个值

函数可以返回多个值，实际返回的是一个 tuple：

```python
def divide(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder

q, r = divide(17, 5)      # q=3, r=2

# 也可以不解包
result = divide(17, 5)     # result = (3, 2)
```

这个模式在真实项目中非常常见——比如一个函数同时返回数据和错误状态。

---

> 下一篇：[06 - 模块与包管理](./06-modules-and-packages.md)
