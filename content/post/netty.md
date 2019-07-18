---
title: "Java Netty框架源码分析"
tags: ["java", "Netty", "source code analysis"]
date: 2019-7-09
draft: false
---

### 两个问题
- 服务端的socket在哪里启动
  - 反射创建服务端channel. New Socket() 通过jdk创建底层jdk channel.
  - 创建NioServerSocketConfig()用来配置TCP参数
  - AbstractNioChannel()
    - configure(false)配置channel为非阻塞
    - AbstractChannel() 创建id(唯一标识符), unsafe(底层方法), pipeline(逻辑链).
- 在哪里accept连接

#### netty服务端的启动

- 创建服务端Channel
  - bind(): 用户代码接口
    - initAndRegister(): 初始化并注册
    - newChannel(): 创建服务端Channel
- 初始化服务端Channel
- 注册selector
- 端口绑定


##### 创建服务端Channel

- 反射创建服务端channel. New Socket() 通过jdk创建底层jdk channel.
- 创建NioServerSocketConfig()用来配置TCP参数
- AbstractNioChannel()
  - configure(false)配置channel为非阻塞
  - AbstractChannel() 创建id(唯一标识符), unsafe(底层方法), pipeline(逻辑链).

##### 初始化服务端Channel

- init() 初始化的入口
  - set ChannelOptions, ChannelAttrs
  - set ChildOptions, ChildAtrrs
  - config handler 配置服务端pipeline
  - add ServerBootstrapAcceptor 用于接收到accept()的请求时分配一个nio的服务端的线程.

总的来说, 初始化的功能就是保存用户的配置, 并用其建立一个连接器.

##### 注册Selector

- AbstractChannel.register(channel) 入口方法
  - this.eventLoop = eventLoop 绑定线程
  - register(0) 实际注册
    - doRegister() 调用jdk底层注册
    - invokeHandlerAddedIfNeeded()
    - fireChannelRegistered() 把注册成功的结果传回到用户代码

##### 端口绑定

- AbstractUnsafe.bind 入口
  - doBind()
    - javaChannel.bind() 调用jdk底层绑定
  - pipeline.fireChannelActive() 传播事件
    - HeadContext.readIfIsAutoRead() channel的read事件


### NioEventLoop

三个问题.
- 默认情况下, netty起多少线程,何时启动
- Netty是如何解决jdk空轮询bug的
- Netty如何保证异步串行无锁化

NioEventLoop会在NioEventLoopGroup创建时被创建,如果没有额外的参数, 新建2*cpu核数个线程

- new EventLoopGroup() 线程组
  - new ThreadTaskExecutor()
  - for(){new child()} 构造NioEventLoop, 并指定必要的构造参数

#### ThreadPerTaskExecutor

- 每次执行任务都会创建一个线程实体
- NioEventLoop线程的命名规则为nioEventLoop-1-xx

#### new chiled()

- 保存线程执行器ThreadEventPerTaskExecutor
- 创建一个MpscQueue
- 创建一个selector

#### chooserFactory.newChooser()

- isPowerOfTwo() 判断是否是2的幂
  - PowerOfTwoEventExecutorChooser 进行优化
    - index++ & (length-1)
  - GenericEventExecutorChooser
    - abs(index++ % length)


#### NioEventLoop启动触发器

- 服务端启动绑定端口
  - bind() -> Executor(task) 入口
    - startThread() -> doStartThread() 创建线程
      - ThreadPerTaskExecutor.execute()
        - thread=Thread.currentThread()
        - NioEventLoop.run()启动
- 新连接接入通过chooser绑定一个NioEventLoop

#### NioEventLoop.run()

- run() -> for(;;)
  - select() 检查是否有IO事件
  - processSelectedKeys() 处理IO事件
  - runAllTasks() 处理异步任务队列

##### select()方法执行逻辑

- deadlock 以及任务穿插逻辑处理
- 阻塞式select
- 避免jdk空轮询的bug

##### processSelectedKey()执行逻辑

- selected keySet优化
- processSelectedKeysOptimized()

##### runAllTasks()

- task的分类和添加
- 任务的聚合
- 任务的执行
