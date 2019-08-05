---
title : "深入理解java网络开发知识的归纳"
date: 2019-06-04T01:37:56+08:00
lastmod: 2019-04-14T01:37:56+08:00
draft: false
tags: ["Java", "Web"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---
## 计算机网络
### OSI 七层模型

- 应用层(Application Layer): 文件传输, 电子邮件, 文件服务, 虚拟终端
- 表示层(Presentation Layer): 信息的语法语义以及它们的关联, 如加密解密, 转化翻译, 压缩解压缩.
- 会话层(Session Layer): 不同机器上的用户之间建立及管理会话
- 传输层(Transport Layer): 接受上一层的数据, 在必要的时候把数据进行分割, 并将这些数据交给网络层, 且保证这些数据段有效到达对端
- 网络层(Network Layer): 控制子网的运行, 如逻辑编址, 分组传输, 路由选择
- 数据链路层(Linked Layer): 物理寻址, 同时将原始比特流转变为逻辑传输线路
- 物理层(Physical Layer): 机械,电子,定时接口通信信道上的原始比特流传输.

#### TCP相关
IP协议是无连接的通信协议,他不会占用正在通信的两个计算机的通信线路, IP极大降低了对网络线路的需求. IP的作用仅仅是将每个包路由至他的目的地. 它没有机制去检查或保证数据包的顺序性和准确性.

TCP(Transmission Control Protocol) 传输控制协议简介

- 面向连接的,可靠的,基于字节流的传输层通信协议
- 将应用层的数据流分割成报文段并发送给目标节点的TCP层.
- 数据包都有序号,对方收到则发送ACK确认,未收到则重传.使用校验和来检验数据在传输过程中是否有误.

> 两台电脑的两个进程能互相通信的最基本的要求是能唯一表示一个进程, 通过这个唯一标识可以找到对应的线程. 我们可以在传输层中使用协议端口号, IP可以唯一标识一个主机, TCP协议和端口号可以唯一标识一个进程. IP+TCP+PORT这种唯一标识的模式也被称为套接字(socket). 也就是说我们只需要把数据传送到某一个合适的端口, 剩下的过程由TCP来负责

![](/media/posts/socket.png)
socket通信题
![](/media/posts/socket1.png)
##### TCP Flags
- URG: 紧急指针标志
- **ACK**: 确认序号标志
- PSH: push标志
- RST: 重置连接标志
- **SYN**: 同步序号, 用于建立连接的过程
- **FIN**: finish标志, 用于释放连接

##### 三次握手过程
![](/media/posts/threehandshake.png)

- 第一次握手: 建立连接时, 客户端发送SYN包(seq=x)到服务器, 并进入SYN_SEND状态, 等待服务器确认.
- 第二次握手: 服务器收到SYN包, 必须确认客户的SYN(ack=x+1), 同时自己也发送一个SYN包(seq=y), 即SYN+ACK包,此时服务器进入SYN_RECV状态.
- 第三次握手: 客户端收到服务器的SYN+ACK包, 此时服务器进入SYN_RECV状态.

##### 四次挥手过程
![](/media/posts/fourshake.png)
- 第一次挥手: Client发送一个FIN, 用来关闭Client到Server的数据传送,Client进入FIN_WAIT_1状态:
- 第二次挥手: Server收到FIN后, 发送一个ACK给Client, 确认序号为收到序号+1(与SYN相同, 一个FIN占用一个序号), Server进入CLOSE_WAIT状态
- 第三次挥手: Server发送一个FIN,用来关闭Server到Client的数据传送, Server进入LAST_ACK状态 
- 第四次挥手: Client收到FIN后, Client进入TIME_WAIT状态,接着发送一个ACK给Server,确认序号为收到序号+1, Server进入CLOSED状态, 完成四次挥手.
  - 确保有足够的时间让对方收到ACK包
  - 避免新旧连接混淆

Linux查看连接数:
```bash
netstat -n | awk '/^tcp/{++S[$NF]}END{for(a in S) print a,S[a]}'
```
##### 滑窗(Sliding window)规则
- RTT(Round Transimission Time): 发送一个数据包到收到对应的ACK, 所花费的时间
- RTO(Retranm): 重传时间间隔

TCP使用滑动窗口做流量控制与乱序重排

- 保证TCP的可靠性
- 保证TCP的流控特性

![](/media/posts/slidingwindow.png)

AdvertisedWindow = MaxRcvBuffer - (LastByteRcvd - LastByteRead)

EffectiveWindow = AdvertisedWindow - (LastByteSent - LastByteAcked)

#### UDP
UDP的特点:

- 面向非连接
- 不维护连接状态, 支持同时向多个客户端传输相同的消息
- 数据包报头只有8个字节,额外开销较小.
- 吞吐量只受限于数据生成速率,传输速率以及机器性能
- 尽最大努力交付, 不保证可靠交付, 不需要维持复杂的链接状态表
- 面向报文,不对应用程序提交的报文信息进行拆分或者合并

#### HTTP(Hyper text transmission protocol)
特点:

- 支持客户/服务器模式
- 简单快速
- 灵活
- 无连接

请求/相应的步骤:

- 客户端连接到Web服务器
- 发送HTTP请求
- 服务器接收请求并返回HTTP响应
- 释放连接TCP连接
- 哭护短浏览器解析HTML内容

> 在浏览器地址栏键入URL, 按下回车后经历的流程
>
> - DNS解析, 查询到IP
> - 根据IP地址, 和服务器建立TCP连接
> - 浏览器发送HTTP请求
> - 服务器处理请求并返回HTTP报文
> - 浏览器收到HTML解析渲染页面
> - 连接结束

##### HTTP状态码

- 1xx: 指示信息, 表示请求已接收, 需要继续处理
- 2xx: 成功, 表示请求已被成功接收, 理解, 接受
- 3xx: 重定向, 要完成请求必须进行更进一步的操作
- 4xx: 服务端错误, 请求有语法错误或请求无法实现
- 5xx: 服务器端错误, 服务器未能实现合法的请求

##### GET与POST请求的区别
- HTTP报文层面: GET将请求信息放在URL, POST放在报文体中
- 数据库层面: GET符合幂等性和安全性, POST不符合
- 其他层面: GET可以被缓存, 被存储, 而POST不行

##### Cookie和Session的区别
**Cookie**

- Cookie是由服务器发送给客户端的特殊信息, 以文本的形式存放在客户端 (RESPONSE_HEADER).
- 客户端再次请求的时候, 会把Cookie回发
- 服务器接收到后, 会解析Cookie生成与客户端相对应的内容

![](/media/posts/cookie.png)

**Session**
- 服务器端的机制, 在服务器上保存的信息.
- 解析客户端请求并操作session id, 按需保存状态信息.

session的实现方式:

- 使用Cookie来实现
- 使用URL会写来实现

Cookie和Session的区别

- Cookie数据存放在客户的浏览器上
- Session相对于Cookie更安全
- 若考虑减轻服务器负担, 应当使用Cookie
## JSP and Serverlet

## Spring Family
### Spring Framework
#### IOC(Inverse of Control)
IOC是Spring Core最核心的部分. 在理解IOC原理之前需要先了解DI(Dependency Inversion)
##### DI(Dependency Inversion)
例如实现 行李箱类

![reactor model](/media/posts/depen.png)

```Java
public class Luggage {
    private Framework framework;
    Luggage(int size){
        this.framework = new Framework(size);
    }

    public void move(){}
} 

public class Framework {
    private Bottom bottom;
    Framework(int size){
        this.bottom = new Bottom(size);
    }
}

public class Bottom {
    private Tire tire;
    Bottom(int size){
        this.tire = new Tire(size);
    }
}

public class Tire {
    private int size;
    Tire(int size){
        this.size = size;
    }
}


 psvm{
     Luggage luggage = new Luggage();
     luggage.move() 
 }
```
这种实现方法基于上层建筑依赖下层建筑的设计模式, 这种模式基本是不可维护的.

我们需要用依赖注入的思想, 把底层类作为参数传递给上层类, 实现上层对下层的控制.
![reactor model](/media/posts/injec.png)
```Java
public class Luggage {
    private Framework framework;
    Luggage(Framework framework){
        this.framework = framework;
    }

    public void move(){}
} 

public class Framework {
    private Bottom bottom;
    Framework(Bottom bottom){
        this.bottom = bottom;
    }
}

public class Bottom {
    private Tire tire;
    Bottom(Tire tire){
        this.tire = tire;
    }
}

public class Tire {
    private int size;
    Tire(int size){
        this.size = size;
    }
}


 psvm{
     Luggage luggage = new Luggage();
     luggage.move() 
 }
```
简单的说就是上层对象不依赖下层的构造参数,而应该将所依赖的下层的对象的实例注入进上层的参数中.

![reactor model](/media/posts/iocdidl.png)
依赖注入可以分为四种:

- Set注入(Setter injection)
- 接口注入(Interface injection)
- 注解注入(annotation injection)
- 构造器注入(construction injection)

![reactor model](/media/posts/ioc.png)

> 依赖倒置原则(Dependency Inversion Principle): 高层模块不应依赖底层模块, 两者都应依赖其抽象.

控制反转容器(Inverse of Control Container)管理Bean的生命周期.

IoC容器的优势

- 避免在各处使用new来创建类,可以做到统一维护. 只需要维护一个configuration.
- 创建实例时不需要了解其中的细节.

##### Spring IOC 
![reactor model](/media/posts/springioc.png)

Spring IOC支持的功能

- 依赖注入
- 依赖检查
- 自动装配
- 支持集合
- 指定初始化方法和销毁方法
- 支持回调某些方法(略带侵入性)

Spring IOC容器核心接口

- BeanFactory
- ApplicationContext
#### Bean
##### BeanDefinition
BeanDefinition主要用来描述Bean的含义, Spring容器在启动的时候会将xml文件里的或者注解里的Bean的定义解析成Spring内部的BeanDefinition.
##### BeanDefinitionRegistry
BeanDefinitionRegistry提供向IoC容器手工注册BeanDefinition对象的方法.
##### BeanFactory: Spring框架最核心的部分

- 提供IOC的配置机制
- 包含Bean的各种定义,便于实例化Bean
- 建立Bean之间的依赖关系
- Bean生命周期的控制

![reactor model](/media/posts/beanfac.png)

BeanFactory与ApplicationContext的比较

- BeanFactory是Spring框架的基础设施,面向Spring
- ApplicationContext面向使用Spring框架的开发者
  - ApplicationContext的功能(继承多个接口)
    - BeanFactory:能够管理,装配Bean
    - ResourcePatternResolver:能够加载资源文件
    - MessageSource:能够实现国际化等功能
    - ApplicationEventPublisher:能够注册监听器,实现监听机制

##### getBean方法的代码逻辑

- 转换beanName
- 从缓存中加载实例
- 实例化Bean
- 检测parentBeanFactory
- 初始化依赖的Bean
- 创建Bean

##### Spring Bean的五个作用域
- singleton: Spring的默认作用域, 容器里拥有唯一的Bean实例
- prototype: 针对每个getBean请求, 容器都会创建一个Bean实例
- request: 会为每个HTTP请求创建一个Bean实例
- session: 会为每个session创建一个Bean实例
- globalSession: 会为每个全局Http Session创建一个Bean实例, 该作用域仅对Portlet有效

##### Spring Bean的生命周期
-  创建过程:
   -  实例化bean
   -  Aware(注入 ID, BeanFactory和AppCtx)
   -  BeanPostProcessor.postProcessBeforeInitialization
   -  InitializingBean.afterPropertiesSet
   -  定制的Bean init方法
   -  BeanPostPorcessor.postProcessAfterInitialization
   -  Bean初始化完毕
-  销毁过程:
   -  若实现了DisposableBean接口,则会调用destroy方法
   -  若配置了destry-method属性,则会调用其配置的销毁方法.


#### AOP(Aspect Oriented Programming)

- 本质上是一种关注点分离(不同的问题交给不同的部分解决)的技术.
- 通用化功能代码的实现, 对应的就是所谓的切面(Aspect)
- 业务功能代码和切面代码分开后, 架构会变的高内聚和低耦合.
- 确保功能的完整性: 切面最终需要被合并到业务中(Weave)

##### AOP的主要名词

- Aspect: 通用功能的代码实现
- Target: 被织入Aspect的对象
- Join Point: 可以作为切入点的机会,所有方法都可以作为切入点
- Pointcut: Aspect实际被应用在的Join Point, 支持正则
- Advice: 类里的方法以及这个方法如何织入到目标方法的方式
  - 前置通知(Before)
  - 后置通知(AfterReturning)
  - 异常通知(AfterThrowing)
  - 最终通知(After)
  - 环绕通知(Around)
- WeAVing: AOP的实现过程

##### AOP的三种织入方式
- 编译时织入:需要特殊的Java编译器, 如AspectJ
- 类加载时织入:需要特殊的Java编译器, 如AspectJ和AspectWerkz
- 运行时织入:Spring采用的方式,通过动态代理的方式, 实现简单

```Java
@Aspect
public class LoggingAspect {
    @Before("execution(public String getName())")
    public void LoggingAdvice(){
        System.out.println("LOG: Get method called.");
    }
}
```
通过一个Aspect方法可以起到任何类调用他们自己的public String getName()方法之前都会调用LoggingAdvice()方法.

我们也可以创造一个切点,应用于其他的AOP方法
```Java
@Aspect
public class LoggingAspect{
    //切点执行前执行
    @Before("allGetters()")
    public void LoggingAdvice() {
        System.out.println("Advice run. Get Method called");
    }
    //切点执行后执行
    @After("allGetters()")
    public void LoggingAdvice() {
        System.out.println("Advice run. Get Method called");
    }
    //切点方法执行完成返回后再执行, 同时可以通过执行返回类型来限制
    @AfterReturning(pointcut="allGetters()", returning="returnString")
    public void LoggingAdvice() {
        System.out.println("Advice run. Get Method called");
    }
    //切点方法执行返回异常后再执行
    @AfterThrowing("allGetters()")
    public void LoggingAdvice() {
        System.out.println("Advice run. Get Method called");
    }
    //在切点方法的前后执行相应的语句,切点执行的地方是proceedingJoinPoint.proceed();
    @Around("allGetters()")
    public void LoggingAdvice(ProceedingJoinPoint proceedingJoinPoint) {
        try{
            System.out.println("Before Advice");
            returnValue = proceedingJoinPoint.proceed();
            System.out.println("After Advice");
        }catch(Throwable e){
            System.out.println("After throwing");
        } finally{
            System.out.println("After finally");
        }
    }
    //用注解来标记哪些类需要执行Loggable操作
    @Around("@annotation(com.yzzhan.java.aspect.Loggable)")
    public void LoggingAdvice(ProceedingJoinPoint proceedingJoinPoint) {
        try{
            System.out.println("Before Advice");
            returnValue = proceedingJoinPoint.proceed();
            System.out.println("After Advice");
        }catch(Throwable e){
            System.out.println("After throwing");
        } finally{
            System.out.println("After finally");
        }
    }

    //切点是所有名字相关的方法
    @Pointcut("execution(* get*())")
    public void allGetters(){}
    //切点是所有参数相关的方法
    @Pointcut("args(name)")
    public void stringArgumentMethods(String name) {
        System.out.println("A method that takes String arg has been called, The value is" + name);
    }
    //切点是类里所有的方法
    @Pointcut("execution(* * com.yzzhan.java.model.Circle.*(..))")
    public void allCircleMethods(){}
}
```
表示类里所有的方法也可以写作
```Java
    @Pointcut("within(com.yzzhan.java.model.Circle"))
    public void allCircleMethods(){}
```

对于切点的信息,我们可以通过传入JoinPoint joinPoint参数,来使用切点的相关数据.


##### Spring AOP的原理
AOP的实现: JdkProxy和Cglib(Code generation lib)

- 由AopProxyFactory根据AdvisedSupport对象的配置来决定
- 默认策略如果目标是接口,则用JdkProxy来实现,否则使用Cglib
- JDKProxy的核心: InvocationHandler接口和Proxy类
  - 通过Java内部的反射机制来实现
  - 反射机制在生成类的过程中比较高效
- Cglib:以继承的方式动态生成目标类代理
  - 借助ASM实现
  - ASM在生成类之后的执行过程比较高效

###### 代理模式
代理模式是由 接口, 真实实现类, 代理类三部分组成
#### Data Support
##### JDBC-DAO-POJO
### Spring事务
#### ACID
#### 隔离级别
#### 事务传播
### SpringMVC
Spring MVC工作流程

1）浏览器送请求至前端控制器DispatcherServlet。
2）DispatcherServlet收到请求调用HandlerMapping处理器映射器。
3）处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4）DispatcherServlet调用HandlerAdapter处理器适配器。
5）HandlerAdapter执行具体的Handler（controller）返回ModelAndView给DispatcherServlet。
6）DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
7）ViewReslover解析后返回具体View。
8） DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。

### Spring Cloud
### Servlet 
![](/media/posts/servlet.png)