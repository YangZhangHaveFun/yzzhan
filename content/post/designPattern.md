---
title: "设计原则与设计模式归纳"
tags: ["java", "design pattern", "UML"]
date: 2019-7-24
draft: false
---

### 开闭原则
一个软件实体如类, 模块和函数都应该对扩展开放,对修改关闭.
- 用抽象构建框架, 用实现扩展细节
- 优点: 提高软件系统的可复用性及可维护性

### 依赖倒置原则
高层模块不应该依赖低层模块, 两者都应该依赖其抽象
- 抽象不应该依赖细节; 细节应该依赖抽象
- 针对接口编程,不要针对实现编程
- 优点: 可以减少类间的耦合性, 提高系统稳定性, 提高代码可读性和可维护性, 可降低修改程序所造成的风险  

### 单一职责原则
不要存在多于一个导致类变更的原因
- 一个类/接口/方法只负责一项职责
- 优点: 降低类的复杂度, 提高类的可读性, 提高系统的可维护性, 降低变更引起的风险.

### 接口隔离原则
用多个专门的接口, 而不使用单一的总接口, 客户端不应该依赖它不需要的接口
- 优点: 符合我们常说的高内聚低耦合的设计思想. 从而使得类具有很好的可读性, 可扩展性和可维护性

### 迪米特原则
一个对象应该对其他对象保持最少的了解. 又叫最少知道原则
- 尽量降低类与类之间的耦合

## 创建型
### 简单工厂模式
### 工厂模式
定义一个创建对象的接口但让实现这个接口的类来决定实例化哪个类, 工厂方法让类的实例化推迟到子类中进行

- 适用场景: 
  - 创建对象需要大量重复的代码
  - 客户端(应用层)不依赖于产品类实例如何被创建,实现等细节
  - 一个类通过子类来制定创建对象
- 优点:
  - 用户只需要关心所需要产品对应的工厂,无须关心创建细节
  - 加入新产品符合开闭原则,提高可扩展性
- 缺点:
  - 类的个数容易过多, 增加复杂度
  - 增加了系统的抽象性和理解难度

###抽象工厂
客户端(应用层)不依赖于产品类实例如何被创建,实现等细节

- 强调一系列相关的产品对象(属于同一产品族)一起使用创建对象需要大量重复的代码
- 提供一个产品类的库,所有的产品以同样的接口出现,从而使客户端不依赖于具体实现
- 优点: 
  - 具体产品在应用层代码隔离, 无须关心创建细节
  - 将一个系列的产品族同一到一起创建
- 缺点:
  - 规定了所有可能被创建的产品集合, 产品族中扩展新的产品困难,需要修改抽象工厂的接口
  - 增加了系统的抽象性和理解难度

### 建造者模式
将一个复杂对象的构建与它的表示分离, 使得同样的构建过程可以创建不同的表示.

- 用户只需指定需要建造的类型就可以得到他们, 建造过程及细节不需要知道
- 适用场景: 
  - 如果一个对象有非常复杂的内部结构(很多的属性)
  - 想把复杂对象的创建和适用分离
- 优点: 
  - 封装性好, 创建和使用分离
  - 扩展性好, 建造类之间独立, 一定程度上解耦
- 缺点:
  - 产生多于的Builder对象
  - 产品内部发生变化,建造者都要修改,成本较大

### 单例模式
保证一个类仅有一个实例,并提供一个全局访问点

- 想确保任何情况下都绝对只有一个实例
- 优点:
  - 在内存里只有一个实例,减少了内存的开销
  - 可以避免对资源的多重占用
  - 设置全局访问点,严格控制访问
- 缺点:
  - 没有借口,扩展困难
- 重点:
  - 私有构造器
  - 线程安全
  - 延迟加载
  - 序列化和反序列化
  - 反射

#### LazySingleton
懒汉模式的优点是可以延时生成实例. 
```Java
public class LazySingleton {
    private static LazySingleton lazySingleton = null;

    private LazySingleton(){

    }

    public static LazySingleton getInstance(){
        if(lazySingleton == null){
            lazySingleton = new LazySingleton();
        }
        return lazySingleton;
    }
}
```
缺点是存在并发问题. 有两种解决方案
![](media/posts/singletondc.png)
一种从可见性方面提供解决方案, 使其他线程不能发现指令重排
##### Double Check
```Java
public class DoubleCheckLazySingleton {
    private static DoubleCheckLazySingleton lazySingleton = null;

    private DoubleCheckLazySingleton(){

    }

    public static DoubleCheckLazySingleton getInstance(){
        if(lazySingleton == null){
            synchronized(DoubleCheckLazySingleton.class){
                if(lazySingleton == null){
                    lazySingleton = new LazySingleton();
                }
            }
        }
        return lazySingleton;
    }
}
```
##### 内部类加载机制实现懒汉模式
![](media/posts/innerclass.png)
```Java
public class StaticInnerClassLazySingleton {
    private static class InnerClass{
        private static StaticInnerClassLazySingleton staticInnerClassLazySingleton();
    }
    public static StaticInnerClassLazySingleton getInstance(){
        return InnerClass.staticInnerClassLazySingleton;
    }
}
```
#### 单例模式和工厂模式
#### 单例模式和享元模式
