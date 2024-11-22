# 使用 asyncio 包处理并发

`asyncio` 包，这个包使用事件循环驱动的协程实现并发。
`asyncio` 包支持 Python 3.3 及以上版本，并在 Python 3.4 中正式成为标准库。

## 协程的优点

多线程的代码，很容易在任务执行过程中被挂起；而协程的代码是手动挂起的，只要代码没有运行到 `await`，调度器就不会挂起这个协程。

In [ ]:

```
# 基于 asyncio 的协程并发实现
# https://github.com/fluentpython/example-code/blob/master/18b-async-await/spinner_await.py
# 在 Jupyter notebook 中运行该代码会报错，所以考虑把它复制出去单独运行。
# 见 https://github.com/jupyter/notebook/issues/3397
import asyncio
import itertools
import sys


async def spin(msg):
    write, flush = sys.stdout.write, sys.stdout.flush
    for char in itertools.cycle('|/-\\'):
        status = char + ' ' + msg
        write(status)
        flush()
        write('\x08' * len(status))
        try:
            await asyncio.sleep(.1)
        except asyncio.CancelledError:
            break
    write(' ' * len(status) + '\x08' * len(status))   # 使用空格和 '\r' 清空本行


async def slow_function():
    # pretend waiting a long time for I/O
    await asyncio.sleep(3)
    return 42


async def supervisor():
    spinner = asyncio.ensure_future(spin('thinking!'))    # 执行它，但不需要等待它返回结果
    print('spinner object:', spinner)
    result = await slow_function()                        # 挂起当前协程，等待 slow_function 完成
    spinner.cancel()                                      # 从 await 中抛出 CancelledError，终止协程
    return result


def main():
    loop = asyncio.get_event_loop()
    result = loop.run_until_complete(supervisor())
    loop.close()
    print('Answer:', result)


main()
```

## 基于协程的并发编程用法

定义协程：

1. 使用 `async def` 定义协程任务；
2. 在协程中使用 `await` 挂起当前协程，唤起另一个协程，并等待它返回结果；
3. 处理完毕后，使用 `return` 返回当前协程的结果

运行协程：不要使用 `next` 或 `.send()` 来操控协程，而是把它交给 `event_loop` 去完成。

```
loop = asyncio.get_event_loop()
result = loop.run_until_complete(supervisor())
```

注：`@asyncio.coroutine` 并**不会预激协程**。

## 使用协程进行下载

[aiohttp](https://aiohttp.readthedocs.io/en/stable/) 库提供了基于协程的 HTTP 请求功能。
书中提供的并行下载国旗的简单示例可以在[这里](https://github.com/fluentpython/example-code/blob/master/17-futures/countries/flags_await.py)看到。
完整示例在下载国旗的同时还下载了国家的元信息，并考虑了出错处理及并发数量，可以在[这里](https://github.com/fluentpython/example-code/blob/master/17-futures/countries/flags3_asyncio.py)看到。

`asyncio.Semaphore` 提供了协程层面的[信号量](https://zh.wikipedia.org/zh-cn/信号量)服务，我们可以使用这个信号量来[限制并发数](https://github.com/fluentpython/example-code/blob/master/17-futures/countries/flags3_asyncio.py#L61)。

```
with (await semaphore):                   # semaphore.acquire
    image = await get_flag(base_url, cc)
                                          # semaphore.release
```

## 在协程中避免阻塞任务

文件 IO 是一个非常耗时的操作，但 asyncio 并没有提供基于协程的文件操作，所以我们可以在协程中使用 `run_in_executor` 将任务交给 `Executor` 去执行[异步操作](https://github.com/fluentpython/example-code/blob/master/17-futures/countries/flags3_asyncio.py#L74)。

注：[aiofiles](https://github.com/Tinche/aiofiles) 实现了基于协程的文件操作。

## 使用 aiohttp 编写 Web 服务器

廖雪峰写了个关于 asyncio 的[小教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014320981492785ba33cc96c524223b2ea4e444077708d000)。
如果需要实现大一点的应用，可以考虑使用 [Sanic](https://sanic.readthedocs.io/en/latest/) 框架。基于这个框架的 Web 应用写法和 Flask 非常相似。

## 使用 asyncio 做很多很多事情

GitHub 的 [aio-libs](https://github.com/aio-libs) 组织提供了很多异步驱动，比如 MySQL, Zipkin 等。