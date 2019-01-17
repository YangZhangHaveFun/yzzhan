---
title : "实现高并发并深入理解asyncio内置包"
date: 2019-01-16T01:37:56+08:00
lastmod: 2019-01-16T01:37:56+08:00
draft: false
tags: ["Python", "Overview"]
categories: ["Advanced Python Theory"]
author: "Yang Zhang"

---
## Package Asynic

实现协程和io多路复用，python提供了一个内置包---asyncio去简化 回调+事件循环(核心)+select(poll,epoll)这个流程。

在async包的内部，已经实现了事件循环，并且为了把generator和coroutine方法区分开，python引入了两个关键词 async和await

-async用于方法开头，声明此方法为协程方法

-await用于方法中，声明需要异步处理的地方，类似于生成器里的yield from

-同时方法末尾可提供一个返回值。

-在协程方法中不可以使用同步/阻塞的方法或函数(eg: time.sleep() --->  async.sleep())，否则会使异步编程无意义

## Adding tasks into loop

1. 我们实例化loop ---> loop = asyncio.get_event_loop()

2. 然后列出需要异步实现的Tasks. 此时我们有两种实现方法。第一我们可以把我们的task转成Future object eg: 
```python 
futures = [asyncio.ensure_future(get_html("www.baidu.com/{}".format(i))) for i in range(100)]`

for future in futures:
    future.add_done_callback(callbakck)

```
这个方法与线程池的实现方法和原理相同，在future object里存着返回的值，并用result()可以调出。并且，future object支持增加回调函数.当我们的回调函数需要输入参数时，我们需要用到包装函数partial ---- from functools import partial
```python
for future in futures:
    future.add_done_callback(partial(callbakck, "www.baidu.com"))
```

或者用async.wait, async.gather包装
```python 
    task_group_1 = [get_html("www.bilibili.com/{}".format(i))for i in range(10)]
    task_group_2 = [get_html("www.imooc.com/{}".format(i)) for i in range(10)]

    results = loop.run_until_complete(asyncio.gather(*task_group_1, *task_group_2))
```
此时的对象为task object，对象内不会直接存储返回值。如需用到回调函数我们需用future object的方法或者直接使用更底层的方法手动给事件循环添加事件监听
```python 
    task = loop.creat_task(get_html())

    task.add_done_callback(callback)
```
async.gather 方法是对async.wait更高级的包装，支持对task进行分类

3. 最后我们需要用到loop里的run_until_complete()函数，类似于线程池里的as_completed. 里面参数为未来对象，可迭代。

```python
import asyncio
import time
from functools import partial

async def get_html(url):
    print("Start catching the html file.")
    await asyncio.sleep(2)
    print("The html file have been catched  successfully.")
    return url

def callbakck(url,future):
    print("The email has been sent successfully.")
    print(url)


if __name__ == "__main__":
    start_time = time.time()
    loop = asyncio.get_event_loop()

    futures = [asyncio.ensure_future(get_html("www.baidu.com/{}".format(i))) for i in range(100)]
    for future in futures:
        future.add_done_callback(partial(callbakck, "www.baidu.com"))

    loop.run_until_complete(asyncio.wait(futures))

    print([future.result() for future in futures])
    print(time.time() - start_time)
```

```python
import asyncio
import time


async def get_html(url):
    print("Start catching the html file.")
    await asyncio.sleep(2)
    print("The html file have been catched  successfully.")
    return url


def callbakck(future):
    print("The email has been sent successfully.")


if __name__ == "__main__":
    start_time = time.time()
    loop = asyncio.get_event_loop()

    task_group_1 = [get_html("www.bilibili.com/{}".format(i))for i in range(10)]
    task_group_2 = [get_html("www.imooc.com/{}".format(i)) for i in range(10)]

    results = loop.run_until_complete(asyncio.gather(*task_group_1, *task_group_2))

    print(results)

    print(time.time() - start_time)
```