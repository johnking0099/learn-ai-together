> 返回 [README 总览](./README.md)

# 08 - 错误处理与异常

程序出错时，Python 会抛出**异常**（Exception）。学会处理异常，是写健壮代码的基础。

---

## 常见异常类型

```python
# ValueError：值不合法
int("abc")                 # ValueError: invalid literal for int()

# KeyError：字典键不存在
d = {"a": 1}
d["b"]                     # KeyError: 'b'

# TypeError：类型不匹配
"hello" + 42               # TypeError: can only concatenate str to str

# FileNotFoundError：文件不存在
open("不存在的文件.txt")     # FileNotFoundError

# IndexError：索引越界
lst = [1, 2, 3]
lst[10]                    # IndexError: list index out of range

# AttributeError：属性不存在
None.split()               # AttributeError: 'NoneType' has no attribute 'split'
```

初学者不需要背这些——遇到报错时，**先读异常名和消息**，90% 的情况能直接定位问题。

---

## try / except / else / finally

### 基本用法

```python
try:
    number = int(input("请输入数字："))
    result = 100 / number
except ValueError:
    print("输入的不是数字")
except ZeroDivisionError:
    print("不能除以零")
```

### 完整语法

```python
try:
    data = json.loads(raw_text)
except json.JSONDecodeError as e:
    print(f"JSON 解析失败：{e}")
else:
    # 没有异常时执行
    print(f"解析成功：{data}")
finally:
    # 无论是否有异常都会执行（常用于清理资源）
    print("处理完毕")
```

- `except` 后面跟具体的异常类型
- `as e` 可以获取异常对象的详细信息
- `else` 在没有异常时执行
- `finally` 无论如何都执行

---

## 为什么不要裸 except

```python
# 错误写法！
try:
    do_something()
except:          # 裸 except：捕获所有异常，包括 KeyboardInterrupt
    pass         # 静默吞掉错误，调试时会非常痛苦

# 正确写法：捕获具体异常
try:
    do_something()
except ValueError:
    handle_value_error()
except (TypeError, KeyError) as e:
    print(f"出错了：{e}")
```

如果你确实想捕获"大多数"异常，用 `Exception`（不包含 `KeyboardInterrupt` 等系统异常）：

```python
try:
    do_something()
except Exception as e:
    print(f"发生错误：{e}")
```

---

## raise 抛出异常

### 主动抛出

```python
def set_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")
    if age > 150:
        raise ValueError("年龄不合理")
    return age
```

### Re-raise（重新抛出）

捕获异常后做一些处理，然后继续向上传播：

```python
try:
    result = call_external_api()
except ConnectionError as e:
    log.error(f"API 调用失败：{e}")
    raise    # 重新抛出原始异常，让调用方处理
```

---

## 自定义异常

继承 `Exception` 即可：

```python
class UserNotFoundError(Exception):
    pass

class InvalidTokenError(Exception):
    def __init__(self, token):
        super().__init__(f"无效的 token: {token}")
        self.token = token
```

```python
def get_user(user_id):
    user = db.find(user_id)
    if user is None:
        raise UserNotFoundError(f"用户 {user_id} 不存在")
    return user
```

自定义异常让错误更具语义——调用方可以精确捕获特定类型的错误。

---

## 真实模式：API 错误处理

在 FastAPI 项目中（衔接 [02 - Web 框架](../02-web-framework/)），错误处理是这样的：

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    user = db.find(user_id)
    if user is None:
        raise HTTPException(
            status_code=404,
            detail=f"用户 {user_id} 不存在"
        )
    return user
```

`HTTPException` 就是一个自定义异常，FastAPI 框架捕获它后自动返回对应的 HTTP 错误响应。这个模式在 Web 开发中无处不在。

---

> 下一篇：[09 - 文件操作与常用标准库](./09-file-and-io.md)
