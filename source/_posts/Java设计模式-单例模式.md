---
title: Java设计模式-单例模式
date: 2016-10-02 17:58:27
description: "Java设计模式学习笔记"
categories: [单例模式]
tags: [设计模式]
---
# 单例模式
保证一个类仅有一个实例，并提供一个访问它的全局访问点。

>适用性：当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。当这个唯一实例应该是通过子类化可扩展的，并且客户应该无需更改代码就能使用一个扩展的实例时。

## 占位符式

- 线程安全
> JVM内部机制保证在执行类构造器初始化的时候是线程安全的

```
public class JustDemo {
    private  static  boolean flag = true;
    public JustDemo() {
        synchronized (JustDemo.class){
            if(flag){
                flag = false;
            }else{
                throw new RuntimeException("反射入侵已阻止");
            }
        }
    }
    private static class Single{
        private static final JustDemo INSTANCE = new JustDemo();
    }
    public JustDemo getInstance() {
        return Single.INSTANCE;
    }
}
```
> 上面示例可防止Java反射方法调用

## 懒汉式

```
public class LazySingleton {
    private static LazySingleton instance;

    private LazySingleton() {
    }

    public static synchronized LazySingleton getInstance() {
        if (instance == null){
            instance = new LazySingleton();
        }
        return instance;
    }
}
```


## 饿汉式
```
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {
    }
    public static Singleton getInstance() {
            return instance;
    }
}
```

