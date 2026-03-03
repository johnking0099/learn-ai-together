> 返回 [README 总览](./README.md)

# 07 - 类与面向对象

当你需要把数据和操作数据的方法绑定在一起时，就该用类（class）了。

---

## 从 dict 到 class

假设你要表示一个用户：

```python
# 用 dict：简单但松散
user = {"name": "Alice", "age": 25, "email": "alice@example.com"}
user["name"]    # 没有自动补全，拼错键名不会报错
```

```python
# 用 class：结构清晰，IDE 友好
class User:
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

user = User("Alice", 25, "alice@example.com")
user.name       # IDE 自动补全，拼错会有提示
```

---

## 基本语法

```python
class User:
    def __init__(self, name, age):
        self.name = name      # 实例属性
        self.age = age

    def greet(self):          # 实例方法
        return f"你好，我是 {self.name}"

    def is_adult(self):
        return self.age >= 18
```

- `__init__`：构造方法，创建实例时自动调用
- `self`：指向实例本身，**必须是实例方法的第一个参数**（Python 的显式设计）
- 实例属性在 `__init__` 中通过 `self.xxx = xxx` 定义

```python
alice = User("Alice", 25)
print(alice.greet())          # "你好，我是 Alice"
print(alice.is_adult())       # True
```

---

## 方法类型

### @classmethod

类方法，通过类本身调用，第一个参数是 `cls`（类本身）：

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @classmethod
    def from_dict(cls, data):
        return cls(data["name"], data["age"])

# 用法：作为替代构造方法
user = User.from_dict({"name": "Alice", "age": 25})
```

### @staticmethod

静态方法，不需要访问实例或类，只是逻辑上属于这个类：

```python
class MathUtils:
    @staticmethod
    def is_even(n):
        return n % 2 == 0

MathUtils.is_even(4)    # True
```

---

## 继承

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

class Dog(Animal):
    def speak(self):               # 覆写（override）父类方法
        return f"{self.name}: 汪汪！"

class Cat(Animal):
    def __init__(self, name, indoor=True):
        super().__init__(name)     # 调用父类的 __init__
        self.indoor = indoor

    def speak(self):
        return f"{self.name}: 喵～"
```

```python
dog = Dog("旺财")
cat = Cat("咪咪")
print(dog.speak())     # "旺财: 汪汪！"
print(cat.speak())     # "咪咪: 喵～"
```

`super()` 用于调用父类的方法，最常见的场景是在子类 `__init__` 中调用父类的 `__init__`。

---

## @dataclass（重点）

`dataclass` 是 Python 3.7+ 引入的装饰器，**在现代 Python 代码中极其常见**。它自动生成 `__init__`、`__repr__` 等方法：

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
    email: str = ""    # 默认值

# 自动生成 __init__，不需要手动写
user = User("Alice", 25, "alice@example.com")
print(user)            # User(name='Alice', age=25, email='alice@example.com')
print(user.name)       # "Alice"
```

对比普通 class：

```python
# 普通 class：需要手动写很多样板代码
class User:
    def __init__(self, name, age, email=""):
        self.name = name
        self.age = age
        self.email = email

    def __repr__(self):
        return f"User(name={self.name!r}, age={self.age!r}, email={self.email!r})"
```

`@dataclass` 帮你省掉了这些重复代码。在 FastAPI、配置管理等场景中你会经常看到它。

---

## 常见魔术方法

以双下划线 `__` 开头和结尾的方法叫 **魔术方法**（magic methods / dunder methods）。看到它们时不要慌——它们是 Python 的特殊协议：

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __str__(self):
        """print() 时调用"""
        return f"{self.name} - ¥{self.price}"

    def __repr__(self):
        """调试/日志时调用"""
        return f"Product({self.name!r}, {self.price})"

    def __len__(self):
        """len() 时调用"""
        return len(self.name)

product = Product("Python 入门", 49.9)
print(product)          # "Python 入门 - ¥49.9"    调用 __str__
len(product)            # 10                       调用 __len__
```

你不需要记住所有魔术方法，只需知道：**`__xxx__` 是 Python 的约定接口**，让你的对象能配合内置函数（`print`、`len`、`in` 等）工作。

---

## 鸭子类型（Duck Typing）

Python 不关心对象的类型，只关心它有没有需要的方法：

> "如果它走路像鸭子、叫起来像鸭子，那它就是鸭子。"

```python
class Duck:
    def quack(self):
        print("嘎嘎！")

class Person:
    def quack(self):
        print("我在学鸭子叫：嘎嘎！")

def make_it_quack(thing):    # 不关心 thing 是什么类型
    thing.quack()             # 只要它有 quack() 方法就行

make_it_quack(Duck())        # "嘎嘎！"
make_it_quack(Person())      # "我在学鸭子叫：嘎嘎！"
```

鸭子类型是 Python 灵活性的核心，也是为什么 Python 代码中很少看到 Java 风格的接口定义。理解这个概念有助于你读懂框架源码中"看起来很松散"的设计。

---

> 下一篇：[08 - 错误处理与异常](./08-error-handling.md)
