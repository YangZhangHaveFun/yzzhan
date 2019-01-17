---
title : "如何在python环境实现并发和一些对并发的理解"
date: 2019-01-16T01:37:56+08:00
lastmod: 2019-01-16T01:37:56+08:00
draft: false
tags: ["Python", "Overview"]
categories: ["Advanced Python Theory"]
author: "Yang Zhang"

---

# 概念： 并发，并行，同步，异步，阻塞， 非阻塞

# 并发的开发 --- 回调 + 事件循环 + select(poll,epoll)

```python
import socket
from urllib.parse import urlparse
from selectors import DefaultSelector, EVENT_READ, EVENT_WRITE


selector = DefaultSelector()
urls = ["https://www.baidu.com", "https://www.baidu.com"]
stop = False


class Fetcher:

    def connected(self, key):
        selector.unregister(key.fd)
        self.client.send("GET {} HTTP/1.1\r\nHost:{}\r\nConnection:close\r\n\r\n".format(self.path,
                                                                                         self.host).encode("utf8"))
        # 当写入成功时，我们继续注册selector用以接受请求的回答
        selector.register(self.client.fileno(), EVENT_READ, self.received)

    def received(self, key):
        d = self.client.recv(1024)
        if d:
            self.data += d
        else:
            selector.unregister(key.fd)
            data = self.data.decode("utf8")
            html_data = data.split("\r\n\r\n")[1]
            print(html_data)
            self.client.close()

            urls.remove(self.next_url)
            if not urls:
                global stop
                stop = True

    def get_url(self, url):
        self.next_url = url
        url = urlparse(url)
        self.host = url.netloc
        self.path = url.path
        self.data = b""
        if self.path == "":
            self.path = "/"

        self.client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.client.setblocking(False)

        try:
            self.client.connect((self.host, 80))
        except BlockingIOError as e:
            pass

        # 接下来最重要的一步，把selector注册到socket里
        selector.register(self.client.fileno(), EVENT_WRITE, self.connected)


# 事件循环，不停请求socket的状态并调用对应的回调函数 ---> 回调 + 事件循环 + select(poll,epoll)
def loop():
    # 1. select本身不支持register方法
    # 2. socket状态变化后的回调方法需自行完成
    while not stop:
        ready = selector.select()
        for key, mask in ready:
            call_back = key.data
            call_back(key)


if __name__ == "__main__":
    fetcher = Fetcher()

    # for url in range(20):
    #     url = "https://stackoverflow.com/questions/{}/".format(url)
    #     urls.append(url)
    #     fetcher = Fetcher()
    #     fetcher.get_url(url)
    # loop()

    for url in urls:
        fetcher = Fetcher()
        fetcher.get_url(url)
    loop()

```

Properties of class means its methods and variables. Properties of instance is similar, which will initialized by **def __init__(self)**. 

## Search order of Properties

The order should be bottom-up. 

![Visual Form](/images/zRHzx.png)

## 并发的优缺点

- advantages:
    1. 可以有效提高单线程的运行效率

- Disadvantages:
    1. 可读性差
    2. 共享状态管理困难
    3. 异常处理困难