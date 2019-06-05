---
title : "深入理解java网络开发知识的归纳"
date: 2019-06-04T01:37:56+08:00
lastmod: 2019-04-14T01:37:56+08:00
draft: false
tags: ["Java", "Web"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---

## JSP and Serverlet

## Spring Family
### Spring Framework
#### Bean
#### AOP

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

#### Data Support
##### JDBC-DAO-POJO

### SpringMVC
### Spring Cloud