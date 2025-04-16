---
sidebar_position: 6
---

# Python协程

## 简要介绍

Python 中的协程是一种用于实现异步编程的机制，它能在单线程内调度和切换任务，从而在 I/O 密集型或高并发场景下有效利用资源。从 Python 3.5 开始，Python 引入了 ```async``` 和 ```await``` 关键字，使得编写异步代码变得更直观。通过在函数前加上 ```async``` 定义协程函数，并使用 ```await``` 等待其他异步操作的完成，从而实现非阻塞式的程序执行。

下面的示例使用了 ```asyncio``` 库，演示如何使用 ```async/await``` 定义协程，并利用事件循环并发执行多个任务：

```python
import asyncio

# 协程函数由 async 定义
async def say_hello():
    print("Hello")
    # 协程操作（async 定义的操作）在其他协程操作中以 await 调用
    await asyncio.sleep(1)
    print("World")

# 定义主协程
async def main():
    # 使用 asyncio.gather 并发运行两个 say_hello 协程
    await asyncio.gather(say_hello(), say_hello())

# 协程操作在非协程操作中以 asyncio.run 调用，该命令会创建一个事件循环并运行 main 协程
asyncio.run(main())
```

