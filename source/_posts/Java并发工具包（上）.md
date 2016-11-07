---
title: Java并发工具包（上）
date: 2016-10-02 17:58:27
description: "Java并发操作中使用的工具类"
categories: [Thread]
tags: [Java并发]
---

有助于创建强大的并发编程
- 同步器：为每种特定的同步问题提供了解决方案
- 执行器：用来管理线程的执行，如线程池
- 并发集合：提供了集合框架中集合的并发版本

## Semaphore
控制一定资源的消费与回收，Semaphore 可以控制某个资源被同时访问的任务数，它通过acquire()获取一个许可，release()释放一个许可。如果被同时访问的任务数已满，则其他 acquire 的任务进入等待状态，直到有一个任务被 release 掉，它才能得到许可。
>特点:经典的信号量，通过计数器控制对共享资源的访问
>常用
- Semaphore(int count):创建拥有count个许可证的信号量
- acquire()/acquire(int num)：获取1/num个信号量
- release()/release(int num):释放1/num个信号量

```
package chapter.concurrent;

import java.util.concurrent.Semaphore;

/**
 * Created by @Just on 2016/10/5.
 */
public class SemaphoreDemo {
    public static void main(String args[]){
        //创建2个信号量，有三个对象要使用
        Semaphore semaphore = new Semaphore(2);
        new Person(semaphore,"A").start();
        new Person(semaphore,"B").start();
        new Person(semaphore,"C").start();
    }
    
    
    
}
class Person extends Thread{
    private Semaphore semaphore;
    public Person(Semaphore semaphore,String name){
        this.semaphore = semaphore;
        setName(name);
    }
    public void run(){
        System.out.println(getName() + " is waiting");
        try {
            //获取信号量
            semaphore.acquire();//默认获取一个
            System.out.println(getName()+" is serving");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(getName()+" is done.");
        semaphore.release();//释放信号量
    }
}
```


## CountDownLatch 同步器
>特点：需要等待几秒后才会触发

>常用
- 必须发生指定数量的事件后才可以继续运行
- CountDownLatch(int count):必须发生count个数量才可以打开锁存器
- await()：等待锁存器
- countDown()：触发事件


```
package chapter.concurrent;

import java.util.concurrent.CountDownLatch;

/**
 * 模拟赛跑，倒数指令
 * Created by @Just on 2016/10/5.
 */
public class CountDownLatchDemo {
    public static void main(String args[]){
        //3 表示有三个事件发生才会执行该线程
        CountDownLatch countDownLatch = new CountDownLatch(3);
        new Racer(countDownLatch, "A").start();
        new Racer(countDownLatch, "B").start();
        new Racer(countDownLatch, "C").start();
        for(int i=0;i<3;i++){
            System.out.println("计数:"+(3-i));
            countDownLatch.countDown();//触发事件
            if(i==2){
                System.out.println("start--");
            }
        }
    }
}
class Racer extends  Thread{
    CountDownLatch countDownLatch;

    public Racer(CountDownLatch countDownLatch, String name) {
        this.countDownLatch = countDownLatch;
        setName(name);
    }

    public void run() {
        try {
            //等待
            countDownLatch.await();
            //执行操作
            System.out.println("执行程序: "+getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
## CyclicBarrier同步器
>特点
- 适用于只有多个线程都达到预定点时才可以继续执行
- CyclicBarrier(int num):等待线程的数量
- CyclicBarrier(int num,Runnable action)：等待线程的数量以及所有线程到达后的操作
- await()：到达临界点后暂停线程


```
package chapter.concurrent;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

/**
 * 模拟斗地主
 * Created by @Just on 2016/10/5.
 */
public class CyclicBarrierDemo {
    public static void main(String args[]) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, new Runnable() {
            @Override
            public void run() {
                System.out.println("Game start");
            }
        });
        new Player(cyclicBarrier,"A").start();
        new Player(cyclicBarrier,"B").start();
        new Player(cyclicBarrier,"C").start();


    }
}
class Player extends Thread{
    CyclicBarrier cyclicBarrier;

    public Player(CyclicBarrier cyclicBarrier, String name) {
        this.cyclicBarrier = cyclicBarrier;
        setName(name);
    }
    public void run(){
        System.out.println(getName()+"等待");
        try {
            cyclicBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }

    }

}
```
## Exchanger 交换器
>特点：简化两个线程间数据的交换

常用方法
- Exchanger<V>:指定进行交换的数据类型
- V exchange(V object):等待线程到达，交换数据


```
package chapter.concurrent;

import java.util.concurrent.Exchanger;

/**
 * Created by @Just on 2016/10/5.
 */
public class ExchangerDemo {
    public static void main(String args[]) {
        Exchanger<String> ex = new Exchanger<>();
        new A(ex).start();
        new B(ex).start();
    }
}
class A extends Thread{
    Exchanger<String> ex;
    public A(Exchanger ex){
        this.ex = ex;
    }
    public void run(){
        String str;
        try {
            str = ex.exchange("hello");
            System.out.println("A ex:"+str);

            str = ex.exchange("this is A");
            System.out.println("A ex:"+ str);

            str = ex.exchange("this is A1");
            System.out.println("A ex:"+ str);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
class B extends Thread{
    Exchanger<String> ex;
    public B(Exchanger ex){
        this.ex = ex;
    }
    public void run(){
        String str;
        try {
            str = ex.exchange("hI");
            System.out.println("B ex:"+str);

            str = ex.exchange("this is B");
            System.out.println("B ex:"+ str);

            str = ex.exchange("this is B1");
            System.out.println("B ex:"+ str);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
## Phaser 
>特点：工作方式与CyclicBarrier类似，但是可以定义多个阶段
>常用:
- Phaser()/Phaser(int num):使用指定0/num个party创建Phaser
- register():注册party
- arriveAndAdvance():到达时等待所有party到达
- arriveAndDeregister()：到达时注销线程自己


```
package chapter.concurrent;

import java.util.concurrent.Phaser;

/**
 * Created by @Just on 2016/10/5.
 */
public class PhaserDemo {
    public static void main(String args[]) {
        Phaser phaser = new Phaser(1);
        System.out.println("starting...");
        //假设每个订单有三道工序
        new Worker(phaser,"worker1").start();//执行等待
        new Worker(phaser, "worker2").start();
        new Worker(phaser,"worker3").start();
        //处理三个订单
        for(int i=0;i<3;i++) {
            //处理完每一个订单都要等待，相当于开始执行下一个订单
            phaser.arriveAndAwaitAdvance();
        }
        //所有订单处理完毕，接触在phaser上注册的所有worker
        phaser.arriveAndDeregister();
        System.out.println("all done...");
    }
}
class Worker extends Thread{
    Phaser phaser;
    public Worker(Phaser phaser,String name){
        this.phaser = phaser;
        setName(name);
        phaser.register();//当当前线程注册到phaser中
    }
    public void run(){
        //模拟三个订单
        for(int i=1;i<=3;i++){
            System.out.println("current order:"+i+":"+getName());
            if(i==3){
                //如果是第三个订单，即所有订单都处理完毕，注销phaser
                phaser.arriveAndDeregister();
            }else{//否则，执行等待
                phaser.arriveAndAwaitAdvance();
            }
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```
