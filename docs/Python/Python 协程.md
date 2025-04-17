---
sidebar_position: 6
---

# Python 协程

## 简要介绍

Python 中的协程是一种用于实现异步编程的机制，它能在单线程内调度和切换任务，从而在 I/O 密集型或高并发场景下有效利用资源。从 Python 3.5 开始，Python 引入了 ```async``` 和 ```await``` 关键字，使得编写异步代码变得更直观。通过在函数前加上 ```async``` 定义协程函数，并使用 ```await``` 等待其他异步操作的完成，从而实现非阻塞式的程序执行。

下面的示例使用了 ```asyncio``` 库，演示如何使用 ```async/await``` 定义协程，并利用事件循环并发执行多个任务：

```python
import asyncio

# 协程函数由 async 定义
async def say_hello():
    print("Hello")
    # 协程操作（async 定义的操作）在其他协程操作中以 await 调用
    await asyncio.sleep(5)
    print("World")

# 定义主协程
async def main():
    # 使用 asyncio.gather 并发运行两个 say_hello 协程
    await asyncio.gather(say_hello(), say_hello())
    print("Done")

# 协程操作在非协程操作中以 asyncio.run 调用，该命令会创建一个事件循环并运行 main 协程
asyncio.run(main())
```

## 几种协程启动方式与差异

1. ```asyncio.run``` 会创建一个事件循环并运行协程，一般用于在非协程代码中启动协程，自此之后其方法中的所有协程均由同一事件循环管理。

2. ```await``` 会等待协程完成，一般用于在协程代码中启动协程，当其被调用时，会等待其参数中的协程完成。

3. ```asyncio.gather``` 会同时发射其参数中的所有协程，使用 ```await``` 会等待所有协程完成。

```python
async def main():
    # 使用 asyncio.gather 并发运行两个 say_hello 协程
    await asyncio.gather(say_hello(), say_hello())
    asyncio.sleep(6)
    print("Done")
```

上述代码的输出结果为：

```
Hello
Hello
(等候5秒)
World
World
(等候6秒)
Done
```

asyncio.gather 同时发射了两个协程，等待它们完成后才会继续执行输出 Done。

4. ```asyncio.create_task``` 会发射协程，但不等待其执行完成，除非由 ```await``` 关键字标记。

```python
async def main():
    # 使用 asyncio.create 启动两个 say_hello 协程
    t1 = asyncio.create_task(say_hello())
    t2 = asyncio.create_task(say_hello())
    asyncio.sleep(6)
    print("Done")
```

上述代码的输出结果为：

```
Hello
Hello
(等候5秒)
World
World
(等候1秒)
Done
```

:::tip[有趣的差异]

事实上，```asyncio.gather```和```asyncio.create_task```存在些许不同，当我们使用以下代码：

```python
import asyncio

# 协程函数由 async 定义
async def say_hello():
    print("Hello")
    await asyncio.sleep(5)
    print("World")

async def main():
    # (1) 使用 asyncio.create_task 启动两个 say_hello 协程
    # t1 = asyncio.create_task(say_hello())
    # t2 = asyncio.create_task(say_hello())
    # await asyncio.sleep(2)
    # print("Done")

    # (2) 使用 asyncio.gather 启动两个 say_hello 协程
    # asyncio.gather(say_hello(), say_hello())
    # await asyncio.sleep(2)
    # print("Done")

asyncio.run(main())
```

如果运行 (1)，则输出结果为：

```
Hello
Hello
Done
```

其主要原因是 ```print("Done")``` 在等待两秒运行完成后结束，该协程启动的所有协程也会一并结束。

如果运行 (2)，则输出结果为：

```
Hello
Hello
Done

_GatheringFuture exception was never retrieved
future: <_GatheringFuture finished exception=CancelledError()>
Traceback (most recent call last):
  ···
asyncio.exceptions.CancelledError
```

区别在于```asyncio.gather```会自动检测并对主协程退出进行报错，```asyncio.create_task```不会。

5. ```asyncio.wait``` 会等待协程完成，一般用于在协程代码中启动协程，当其被调用时，会等待其参数中的协程完成。

```asyncio.wait``` 与 ```asyncio.gather``` 的区别在于，前者会返回一个元组，其中包含两个集合，一个是已经完成的协程，另一个是未完成的协程；后者返回一个Future列表，其中包含所有协程的Task。

:::

:::warning[注意]
```asyncio.run``` 本身是阻塞式的，其后的所有内容会等其内部的协程执行完成之后才会执行。
:::

:::danger[错误的使用]
不要在协程中使用 ```time.sleep```，应当选择```await asyncio.sleep```。前者会暂停整个事件循环，所有协程被暂停。
:::

## 