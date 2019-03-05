---
title : "对多线程的理解和深入理解Future源码"
date: 2019-01-16T01:37:56+08:00
lastmod: 2019-01-16T01:37:56+08:00
draft: false
tags: ["Python", "thread"]
categories: ["Advanced Python Theory"]
author: "Yang Zhang"

---
## Basic Concepts related to thread and process

## Multiple ways to implement the multithreading project in python

## Queue, Lock, RLock, Condition and  

```python
# GIL---global interpreter lock
import threading
from queue import Queue
import time
# Communication between threads
# 1. Share Variables
sum = 0
lock = threading.Lock()


def add_glo(lock):
    global sum
    for i in range(1000000):
        lock.acquire()
        sum += 1
        lock.release()


def dec_glo(lock):
    global sum
    for i in range(1000000):
        lock.acquire()
        sum -= 1
        lock.release()


# 2. Queue
def get_detail_html(queue):
    time.sleep(3)
    while True:
        url = queue.get()
        print("get detailed html started")
        time.sleep(1)
        print("get detailed html ended")
        print(url, "remaining: ", queue.qsize())
        if queue.empty():
            break


def put_detail_page(queue):
    print("get detail page started")
    time.sleep(2)
    for i in range(10):
        queue.put("http://projectsedu.com/{id}".format(id=i))
    print("get detail page ended")


if __name__ == "__main__":

    add_thread = threading.Thread(target=add_glo, args=(lock,))
    dec_thread = threading.Thread(target=dec_glo, args=(lock,))

    add_thread.start()
    dec_thread.start()

    """
    Wait until the thread terminates.
        This blocks the calling thread until the thread whose join() method is
        called terminates -- either normally or through an unhandled exception
        or until the optional timeout occurs.
    """
    without_join = sum
    add_thread.join()
    dec_thread.join()
    with_join = sum
    print("-without join ", without_join, " -with join ", with_join)

    start_time = time.time()
    detail_url_queue = Queue(maxsize=10)

    put_thread = threading.Thread(target=put_detail_page, args=(detail_url_queue,))
    get_thread = threading.Thread(target=get_detail_html, args=(detail_url_queue,))

    put_thread.start()
    get_thread.start()

    detail_url_queue.join()
    put_thread.join()
    get_thread.join()
    print("Cost time is {}".format(time.time()-start_time))


```