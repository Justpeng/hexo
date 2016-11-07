---
title: Java并发工具包(下)
date: 2016-10-02 17:58:27
description: "Java并发操作中使用的工具类"
categories: [Thread]
tags: [Java并发]
---
## 执行器
>特点：用于启动并控制线程执行
>常用API
- 核心接口为 Executor,包含一个execute(Runnable)用于指定被执行的线程
- ExecutorService接口用于控制线程执行和管理线程
- 预定义了如下执行器：
1. ThreadPoolExecutor(线程池)
2. ScheduledThreadPoolExecutor
3. ForkJoinPool


### Callable 与 Future
- Callable<V>:表示具有返回值的线程
- V：表示返回值类型
- call():执行任务
- Future<V>:表示Callable的返回值
- V：返回值类型
- get():获取返回值

```
package chapter.concurrent;

import java.util.concurrent.*;

/**
 * Created by @Just on 2016/10/5.
 */
public class ExecutorDemo {

    public static void main(String args[]) throws ExecutionException, InterruptedException {
        //初始化具有两个固定线程的线程池 
        ExecutorService es = Executors.newFixedThreadPool(2);
        //通过submit方法调用call方法；即放入线程中执行
        Future<Integer> f1 =es.submit(new Method(1,100));
        Future<Integer> f2 = es.submit(new Method(100, 1000));
        Future<Integer> f3 = es.submit(new Method(100, 1000));
        System.out.println("f1:"+f1.get());//获取返回值
        System.out.println("f2:"+f2.get());
        System.out.println("f2:"+f3.get());
        //停止执行器
        es.shutdown();
    }
}

class Method implements Callable<Integer> {
    private int begin;
    private int end;

    public Method(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }
    
    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = begin; i <end; i++) {
            sum+=i;
        }
        Thread.sleep(3000);
        return sum;
    }
}

```
## 锁与原子操作
### 锁
- 为使用synchronized控制对资源访问提供了替代机制
- 基本操纵模型：访问资源之前申请锁，访问完毕后释放锁
- lock/tryLock：申请锁
- unlock：释放锁
- 具体锁实现类：ReentrantLock实现了Lock接口

```
package chapter.concurrent;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Created by @Just on 2016/10/5.
 */
public class LockDemo {
    public static void main(String args[]) {
           new AddData().start();
    }
}

class Data {
    static int i=0;
    static  Lock lock = new ReentrantLock();
    static void operate( ){
        lock.lock();
        i++;
        System.out.println("i:"+i);
        lock.unlock();
    }
}
class AddData extends Thread{
    @Override
    public void run() {
       while (true) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Data.operate();
       }
    }
}


```
### 原子操作
java.util.concurrent.atom包中提供了对原子操作的支持
- 提供了不需要锁以及其他同步机制就可以进行的一些不可中断操作
- 主要操作为：获取，设置，比较