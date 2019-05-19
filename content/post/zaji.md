---
title : "杂记"
date: 2019-04-17T01:37:56+08:00
lastmod: 2019-04-17T01:37:56+08:00
draft: false
tags: ["zaji"]
categories: ["quoto"]
author: "Yang Zhang"
---
老子曰: 死而不亡者寿. 
"'太上有立德，其次有立功，其次有立言'，虽久不废，此之谓不朽。"  ---<<左传>>

为君纠错, 是为忠; 为家擎天, 是为孝; 为子作则, 是为礼; 为法辩白, 是为节.

"为天地立心，为生民立命，为往圣继绝学，为万世开太平”     -- 张载


ThreadPoolExecutor的void execute(Runnable command) 方法, 利用这个方法虽然可以提交任务, 但是却没有办法获取任务的执行结果.

Java通过ThreadPoolExecutor提供的3个submit()方法和1个FutureTask工具类来支持获得任务执行结果的需求
```Java
// 提交 Runnable 任务
Future<?> submit(Runnable task);
// 提交 Callable 任务
<T> Future<T> submit(Callable<T> task);
// 提交 Runnable 任务及结果引用
<T> Future<T> submit(Runnable task, T result);
```

Future接口有五个方法.
- cancle(): 取消任务
- isCancelled(): 判断任务是否已经取消
- isDone(): 判断任务是否已经结束
- get(): 获取任务执行结果(阻塞)
- get(timeout, unit): 获取任务执行结果,并支持超时机制.

submit()方法之间的区别
- 提交Runnable任务submit(Runnable task): 这个方法的参数是一个RUNNABLE接口, Runnable接口的run()方法是没有返回值的,所以submit(Runnable task)这个方法返回的Future仅可以用来断言任务已经结束了,类似于Thread.join().
- 提交Callable任务submit(Callable<T> task): 这个方法的参数是一个Callale接口, 它只有一个call()方法, 并且这个方法是有返回值的,所以这个方法返回的Future对象可以通过调用其get()方法来获取任务的执行结果.
- 提交Runnable任务及结果引用submit(Runnable task, T result): Runnable接口的实现类Task声明了一个有参构造函数Task(Result r), 创建Task对象的时候传入了result对象, 这样就能在类Task的run()方法中对result进行各种操作. result相当于主线程和子线程之间的桥梁, 通过它主子线程可以共享数据.

```Java 
ExecutorService executor = Executor.newFixedThreadPool(1);
//创建Result对象
Result r = new Result();
r.setAAA(a);
//提交任务
Future<Result> future = executor.submit(new Task(r),r);
Result fr = future.get()

class Task implements Runnable{
    Result r;
    //通过构造函数传入result
    Task(Result r){
        this.r = r;
    }
    void run() {
        //可以操作 result
        a = r.getAAA();
        r.setXXX(x);
    }
}
```

#### FutureTask工具类
Future前面提到是个接口,而FutureTask是一个工具类, 这个工具类有两个构造函数.
```Java
FutureTask(Callable<V> callable);
FutureTask(Runnable runnable, V result);
```

FutureTask实现了Runnable和Future接口, 由于其实现了Runnable接口, 所以可以将FutureTask对象作为任务提交给ThreadPoolExecutor执行, 也可以直接被Thread执行; 又因为实现了Future接口, 所以也可以用来获得任务的执行结果.
```Java
// 创建 FutureTask
FutureTask<Integer> futureTask
  = new FutureTask<>(()-> 1+2);
// 创建线程池
ExecutorService es = 
  Executors.newCachedThreadPool();
// 提交 FutureTask 
es.submit(futureTask);
// 获取计算结果
Integer result = futureTask.get();
```

#### 实现"烧水泡茶"程序

![烧水泡茶](/media/posts/boil.png)
一种方式可以分为两个FutureTask----ft1和ft2, 
- ft1完成洗水壶, 烧开水, 泡茶的任务. 
- ft2完成洗茶壶, 洗茶杯, 拿茶叶的任务. 

需要注意的是ft1在执行泡茶任务前,需要等待ft2完成所有任务, 换言之ft1需要在内部引用ft2,在执行泡茶之前, 调用ft2的get()方法实现等待.
```Java
// 创建任务 T2 的 FutureTask
FutureTask<String> ft2
  = new FutureTask<>(new T2Task());
// 创建任务 T1 的 FutureTask
FutureTask<String> ft1
  = new FutureTask<>(new T1Task(ft2));
// 线程 T1 执行任务 ft1
Thread T1 = new Thread(ft1);
T1.start();
// 线程 T2 执行任务 ft2
Thread T2 = new Thread(ft2);
T2.start();
// 等待线程 T1 执行结果
System.out.println(ft1.get());

// T1Task 需要执行的任务：
// 洗水壶、烧开水、泡茶
class T1Task implements Callable<String>{
  FutureTask<String> ft2;
  // T1 任务需要 T2 任务的 FutureTask
  T1Task(FutureTask<String> ft2){
    this.ft2 = ft2;
  }
  @Override
  String call() throws Exception {
    System.out.println("T1: 洗水壶...");
    TimeUnit.SECONDS.sleep(1);
    
    System.out.println("T1: 烧开水...");
    TimeUnit.SECONDS.sleep(15);
    // 获取 T2 线程的茶叶  
    String tf = ft2.get();
    System.out.println("T1: 拿到茶叶:"+tf);

    System.out.println("T1: 泡茶...");
    return " 上茶:" + tf;
  }
}
// T2Task 需要执行的任务:
// 洗茶壶、洗茶杯、拿茶叶
class T2Task implements Callable<String> {
  @Override
  String call() throws Exception {
    System.out.println("T2: 洗茶壶...");
    TimeUnit.SECONDS.sleep(1);

    System.out.println("T2: 洗茶杯...");
    TimeUnit.SECONDS.sleep(2);

    System.out.println("T2: 拿茶叶...");
    TimeUnit.SECONDS.sleep(1);
    return " 龙井 ";
  }
}
// 一次执行结果：
T1: 洗水壶...
T2: 洗茶壶...
T1: 烧开水...
T2: 洗茶杯...
T2: 拿茶叶...
T1: 拿到茶叶: 龙井
T1: 泡茶...
上茶: 龙井

```