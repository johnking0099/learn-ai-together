> 返回 [README 总览](./README.md)

# 02 - 变量与基本数据类型

Python 是动态类型语言——变量不需要声明类型，赋值即创建。

---

## 变量赋值

```python
name = "Alice"          # 字符串
age = 25                # 整数
is_active = True        # 布尔值
```

Python 没有 `let`、`var`、`const` 这类关键字，直接写变量名 = 值。

### 多重赋值

```python
a, b = 1, 2             # a=1, b=2
x = y = z = 0           # 三个变量同时赋值为 0
a, b = b, a              # 交换两个变量（Python 的优雅写法）
```

---

## 数字类型

### int（整数）

```python
count = 42
big_number = 1_000_000   # 下划线分隔，方便阅读，等同于 1000000
```

Python 的整数没有大小限制，不会溢出。

### float（浮点数）

```python
price = 19.99
ratio = 3.14e2           # 科学计数法，等于 314.0
```

### bool（布尔值）

```python
is_valid = True
is_empty = False
```

`True` 和 `False` 首字母大写。布尔值本质上是整数：`True == 1`，`False == 0`。

### 除法运算

```python
10 / 3      # 3.3333...  普通除法，结果是 float
10 // 3     # 3          整除（向下取整）
10 % 3      # 1          取余
2 ** 10     # 1024       幂运算
```

`/` 和 `//` 的区别是初学者常见困惑点，记住：**单斜杠得小数，双斜杠得整数**。

---

## 字符串

### 基本创建

```python
s1 = "hello"              # 双引号
s2 = 'hello'              # 单引号，效果完全一样
s3 = """这是
多行字符串"""               # 三引号，支持换行
```

### 转义字符

```python
path = "C:\\Users\\name"   # \\ 表示一个反斜杠
line = "第一行\n第二行"     # \n 表示换行
raw = r"C:\Users\name"    # r 前缀：原始字符串，不转义
```

### f-string（格式化字符串）

**这是 Python 中最常用的字符串格式化方式**，务必熟练掌握：

```python
name = "Alice"
age = 25

# 基本用法
print(f"我叫 {name}，今年 {age} 岁")

# 表达式
print(f"明年 {age + 1} 岁")

# 调用方法
print(f"大写名字：{name.upper()}")

# 格式化数字
pi = 3.14159
print(f"圆周率：{pi:.2f}")       # 保留 2 位小数：3.14

price = 1234567.89
print(f"价格：{price:,.2f}")     # 千分位分隔：1,234,567.89
```

在真实项目中，f-string 无处不在——日志、错误消息、API 响应构造，都会用到。

### 常用字符串方法

```python
text = "  Hello, World!  "

text.strip()              # "Hello, World!"   去除首尾空白
text.split(",")           # ['  Hello', ' World!  ']  按分隔符拆分
"-".join(["a", "b", "c"]) # "a-b-c"          用分隔符连接列表

"hello.py".startswith("hello")   # True
"hello.py".endswith(".py")       # True
"hello world".replace("world", "Python")  # "hello Python"
```

---

## None

`None` 是 Python 中表示"没有值"的特殊对象，类似其他语言的 `null` / `nil`。

```python
result = None

# 判断是否为 None，用 is（不要用 ==）
if result is None:
    print("没有结果")

if result is not None:
    print(f"结果是 {result}")
```

**惯用法**：始终用 `is None` / `is not None`，而不是 `== None`。这是 Python 社区的强烈共识。

---

## 类型转换

```python
int("42")        # 42        字符串转整数
str(42)          # "42"      整数转字符串
float("3.14")    # 3.14      字符串转浮点数
bool(0)          # False     0 为假
bool("")         # False     空字符串为假
bool([])         # False     空列表为假
bool("hello")    # True      非空为真
```

**"假值"规则**：`0`、`0.0`、`""`（空串）、`[]`（空列表）、`{}`（空字典）、`None` 都是假值（falsy），其余为真值（truthy）。

---

## 命名惯例

```python
user_name = "Alice"       # 变量和函数：snake_case
MAX_RETRY = 3             # 常量：UPPER_CASE
_internal_var = "secret"  # 前缀下划线：暗示"内部使用"（约定，非强制）
__name__                  # 双下划线包围：Python 特殊属性（后续章节详解）
```

Python 社区遵循 [PEP 8](https://peps.python.org/pep-0008/) 风格指南，`snake_case` 是最核心的命名规则。

---

> 下一篇：[03 - 核心数据结构](./03-data-structures.md)
