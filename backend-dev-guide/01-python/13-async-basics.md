> 返回 [README 总览](./README.md)

# 13 - 异步编程基础

本篇的目标是让你**能读懂 async/await 代码**，而非写复杂的异步程序。

---

## 为什么需要并发

程序的瓶颈通常分两种：

- **I/O 密集**：等待网络请求、数据库查询、文件读写（大部分 Web 应用）
- **CPU 密集**：数学计算、图像处理、数据分析

```python
# I/O 密集的例子：调用 3 个 API
result1 = call_api_a()    # 等 1 秒
result2 = call_api_b()    # 等 1 秒
result3 = call_api_c()    # 等 1 秒
# 总共需要 3 秒（串行执行）

# 如果能并发执行，3 个请求可以同时发出
# 总共只需要约 1 秒
```

---

## threading：多线程

适合 **I/O 密集** 任务。多个线程可以在等待 I/O 时切换执行：

```python
import threading
import time

def download(url):
    print(f"开始下载 {url}")
    time.sleep(2)          # 模拟网络请求
    print(f"下载完成 {url}")

# 创建多个线程并发执行
threads = []
for url in ["url1", "url2", "url3"]:
    t = threading.Thread(target=download, args=(url,))
    threads.append(t)
    t.start()

# 等待所有线程完成
for t in threads:
    t.join()
```

### GIL 的存在

Python 有一个全局解释器锁（GIL，Global Interpreter Lock），同一时刻只有一个线程执行 Python 代码。这意味着：

- 多线程对 I/O 密集任务有效（等待 I/O 时会释放 GIL）
- 多线程对 CPU 密集任务**没用**（被 GIL 限制为单线程速度）

你不需要理解 GIL 的内部原理，只需记住：**Python 多线程适合 I/O，不适合 CPU 计算**。

---

## multiprocessing：多进程

适合 **CPU 密集** 任务。每个进程有独立的 GIL，真正的并行执行：

```python
from multiprocessing import Pool

def heavy_compute(n):
    return sum(i * i for i in range(n))

# 用进程池并行计算
with Pool(4) as pool:    # 4 个工作进程
    results = pool.map(heavy_compute, [10**6, 10**6, 10**6, 10**6])
```

何时用多进程：数据处理、科学计算、图像批处理等 CPU 密集场景。

---

## async / await 基础（重点）

`async/await` 是 Python 的**协程**机制，用单线程实现高效的 I/O 并发。这是 FastAPI 等现代框架的基础。

### 基本语法

```python
import asyncio

# 定义异步函数（协程）
async def fetch_data(url):
    print(f"开始请求 {url}")
    await asyncio.sleep(1)       # 模拟网络请求（非阻塞等待）
    print(f"请求完成 {url}")
    return f"数据来自 {url}"

# 运行异步函数
async def main():
    result = await fetch_data("https://api.example.com")
    print(result)

asyncio.run(main())
```

关键概念：

- `async def`：定义一个协程函数
- `await`：暂停当前协程，等待异步操作完成
- `asyncio.run()`：启动事件循环，运行协程

### 并发执行多个任务

```python
import asyncio

async def fetch(name, seconds):
    print(f"{name} 开始")
    await asyncio.sleep(seconds)
    print(f"{name} 完成")
    return f"{name} 的结果"

async def main():
    # gather：并发执行多个协程
    results = await asyncio.gather(
        fetch("任务A", 2),
        fetch("任务B", 1),
        fetch("任务C", 3),
    )
    print(results)    # ["任务A 的结果", "任务B 的结果", "任务C 的结果"]

asyncio.run(main())
# 总耗时约 3 秒（最慢的任务决定总时间），而非 6 秒
```

---

## 为什么 FastAPI 用 async

FastAPI 基于 async/await 构建，一个线程就能处理大量并发请求：

```python
from fastapi import FastAPI
import httpx

app = FastAPI()

@app.get("/data")
async def get_data():
    async with httpx.AsyncClient() as client:
        # 等待外部 API 时，可以处理其他请求
        response = await client.get("https://api.example.com/data")
        return response.json()
```

**事件循环**（Event Loop）的核心思想：

```
请求 A 到达 → 开始查数据库 → 等待中...
请求 B 到达 → 开始查数据库 → 等待中...
请求 A 的数据库返回了 → 处理并响应
请求 C 到达 → 开始查数据库 → 等待中...
请求 B 的数据库返回了 → 处理并响应
...
```

一个线程通过在 I/O 等待时切换任务，实现了高并发。这就是 `await` 的本质——"我在等，先去处理别的"。

---

## 读懂 async 代码

在真实项目中你会遇到这些 async 语法：

### async with

异步的上下文管理器，用于需要异步初始化/清理的资源：

```python
async with httpx.AsyncClient() as client:
    response = await client.get(url)

async with aiofiles.open("data.txt") as f:
    content = await f.read()
```

### async for

异步的 for 循环，用于异步迭代器：

```python
async for chunk in response.aiter_bytes():
    process(chunk)
```

### asyncio.gather()

并发执行多个协程，等待全部完成：

```python
user, orders, settings = await asyncio.gather(
    fetch_user(user_id),
    fetch_orders(user_id),
    fetch_settings(user_id),
)
```

---

## 并发方式总结

| 方式 | 适用场景 | 特点 |
|------|---------|------|
| `threading` | I/O 密集 | 简单直观，受 GIL 限制 |
| `multiprocessing` | CPU 密集 | 真正并行，开销较大 |
| `async/await` | I/O 密集 | 单线程高并发，现代框架首选 |

作为初学者，优先掌握 `async/await`——它是 FastAPI 的基础，也是目前 Python Web 开发的主流选择。

---

> 返回 [README 总览](./README.md) | 本章完结，下一步可以学习 [02 - Web 框架与 API](../02-web-framework/)
