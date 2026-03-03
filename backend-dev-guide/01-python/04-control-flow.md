> 返回 [README 总览](./README.md)

# 04 - 控制流

控制程序执行顺序的语法：条件判断和循环。

---

## if / elif / else

```python
score = 85

if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

Python 用**缩进**（4 个空格）表示代码块，没有花括号 `{}`。

### 三元表达式

一行搞定简单的条件赋值：

```python
status = "成年" if age >= 18 else "未成年"

# 等价于
if age >= 18:
    status = "成年"
else:
    status = "未成年"
```

### 条件判断的简写

```python
# 利用 truthy/falsy 特性
if items:              # 列表非空则为 True
    process(items)

if not name:           # 字符串为空则为 True
    name = "匿名"

# 链式比较
if 0 < age < 150:     # Python 支持数学风格的链式比较
    print("合法年龄")
```

---

## for 循环

### 遍历列表

```python
fruits = ["apple", "banana", "cherry"]

for fruit in fruits:
    print(fruit)
```

### range()

```python
for i in range(5):           # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 8):        # 2, 3, 4, 5, 6, 7
    print(i)

for i in range(0, 10, 2):    # 0, 2, 4, 6, 8（步长为 2）
    print(i)
```

### enumerate()（重点）

同时获取索引和元素，**非常常用**：

```python
fruits = ["apple", "banana", "cherry"]

# 不好的写法
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")

# 推荐写法
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# 指定起始索引
for i, fruit in enumerate(fruits, start=1):
    print(f"{i}: {fruit}")
```

在真实项目中，需要索引时几乎都用 `enumerate()`，而不是 `range(len(...))`。

### zip()（重点）

同时遍历多个序列：

```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]

for name, age in zip(names, ages):
    print(f"{name} 今年 {age} 岁")

# 常见用法：两个列表配对成字典
user_dict = dict(zip(names, ages))
# {"Alice": 25, "Bob": 30, "Charlie": 35}
```

### 遍历字典

```python
config = {"host": "localhost", "port": 8080, "debug": True}

for key, value in config.items():
    print(f"{key} = {value}")
```

---

## while 循环

```python
count = 0
while count < 5:
    print(count)
    count += 1
```

### break 和 continue

```python
# break：立即退出循环
for num in range(100):
    if num > 5:
        break
    print(num)          # 只打印 0-5

# continue：跳过当前迭代，进入下一轮
for num in range(10):
    if num % 2 == 0:
        continue
    print(num)          # 只打印奇数
```

---

## match 语句（Python 3.10+）

结构化模式匹配，类似其他语言的 `switch`，但功能更强：

```python
command = "quit"

match command:
    case "start":
        print("启动")
    case "stop":
        print("停止")
    case "quit" | "exit":
        print("退出")
    case _:
        print("未知命令")
```

`match` 语句在较新的项目中开始出现。你不需要深入学习它的高级模式匹配功能，但看到时要能认出来。

---

## any() 和 all()

对可迭代对象做逻辑判断，写出更简洁的代码：

```python
numbers = [2, 4, 6, 8]

all(n > 0 for n in numbers)    # True，所有元素都大于 0
any(n > 5 for n in numbers)    # True，至少有一个大于 5
all(n % 2 == 0 for n in numbers)  # True，全部是偶数
```

真实场景示例：

```python
# 检查所有必填字段是否都有值
required = ["name", "email", "password"]
form_data = {"name": "Alice", "email": "a@b.com", "password": "123"}

if all(form_data.get(field) for field in required):
    print("表单完整")
```

---

> 下一篇：[05 - 函数](./05-functions.md)
