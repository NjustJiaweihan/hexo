---
title: mock-consumer-helper工具包
date: 2021-12-11 22:38:48
tags: DIY
---

跨团队合作时因为排期不同，往往需要先mock外部接口。如果项目用DDD的代码结构的话，调用外部接口的consumer类一般放在防腐层，接口的成功/失败情况都需要mock，对于失败情况需要记录日志、重试等操作，而这些代码在内部开发阶段往往重复性很高。于是有了这个`mock-consumer-helper`工具包。

<!--more-->

## consumer interface

Api接口负责调用外部接口，并对其返回值做处理；Consumer接口记录日志、重试等通用操作。

```java
public interface Consumer<T, S>{
  
  @Autowire
  private Api<T, S> api;

  /**
   * 打日志，重试等默认操作
   */
  default T consumer(S s){
    T t = api.query(s);
  }
}
```

```java
public class Api<T, S>{
   /**
   * 封装外部API的接口
   */
  T query(S s);
}
```

## @MockConsumer

注解的参数是mock接口的成功概率，用于模拟接口的成功/失败两种情况。

```java
@Documented
@Retation(RetationPolicy.RUNTIME)
@Target(ElementType.Method)
public @interface MockConsumer{
  // mock的接口的成功概率
  String value() default "0.5";
}
```

```java
@Aspect
@Component
public class MockConsumerAspect{
  
  /**
   * mock接口使用 概率失败
   */
  @Around("@annotation(mockConsumer)")
  public Object around(ProcessJointPoint joinPoint, MockConsumer mockConsumer){
    boolean flag = Math.random() >= Double.parseDouble(mockConsumer.value());
    if(flag){
      throw new RuntimeException("MockConsumer ==>> decide fail")
    }
    
    Object proceed = null;
    try{
      proceed = joinPoint.preoceed();
    } catch (Throwable throwable){
      throwable.printStackTrace();
    }
    
    return proceed;
  }
}
```

## use case

Todo
