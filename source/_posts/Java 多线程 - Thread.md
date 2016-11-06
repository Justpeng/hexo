---
title: Java 多线程-Thread
date: 2016-10-02 17:58:27
description: "Java线程的基础知识笔记"
categories: [Thread]
tags: [Java]
top: 2
---
## interrupt() 线程中断方法
>javaDoc
- 若中断的不是自身线程，需要进行权限检查 checkAccess(),有权限才可对目标中断
- 若线程T在执行以下三种情况时，对线程T使用interrupted()方法，T的中断标志变为true，当T捕获异常后，中断标志重新变为false
    1. 执行Object.wait(...)方法或该类join(...),sleep(...)方法
    2. java.nio.channels.InterruptibleChannel的I/O操作，关闭通道
    3. java.nio.channels.Selector，立即返回操作数，并可能带有一个非0值
```
  public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();

        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // Just to set the interrupt flag
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
    }
```
### interrupted 与 isInterrunpted区别
- interrupted() 测试**当前线程**是否已经是中断状态，执行后具有状态标志位清除为false的可能，第二次调用 Thread.interrupted()方法，总是返回 false，除非中断了线程。
- isInterrupted() 测试**目标线程对象**是否已经是中断状态，但不清除状态标识

```
    public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }
    public boolean isInterrupted() {
        return isInterrupted(false);
    }
    private native boolean isInterrupted(boolean ClearInterrupted);
```




>示例：interrupt +isInterrupted
```
public class InterruptedDemo {
    public static void main(String args[]) {
        Thread t = new Thread(new ThreadDemo());
        t.start();

        System.out.println("线程t开始，中断标志："+ t.isInterrupted());//false
        //中断线程
        t.interrupt();
        System.out.println("线程t中断标志:："+ t.isInterrupted());//true
        try {
            //模拟 t 抛出异常后中断标志是否消失
            Thread.sleep(2000);
            System.out.println("线程t处理异常后,中断标志："+ t.isInterrupted());//false
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
class ThreadDemo extends Thread {
    public void run() {
        try {
            System.out.println("开始方法...");
            Thread.sleep(300000);
        } catch (InterruptedException e) {
            System.out.println("线程被中断");
            e.printStackTrace();
        }
        System.out.println("结束方法...");
    }
}
```
> 示例

```
  public static void main(String args[]) {
        System.out.println("A:"+Thread.interrupted()); //false
        Thread.currentThread().interrupt();
        System.out.println("B:"+Thread.interrupted()); //true
        System.out.println("C:"+Thread.interrupted()); //false
        System.out.println("D:"+Thread.interrupted()); //false
    }
```
> 此外：终止线程
- 使用退出标志，是线程正常退出，也就是当run方法完成后线程终止；
- 使用stop方法强行终止线程，但是不推荐使用这个方法，因为stop和suspend及resume一样都是作废过期的方法，使用它们可能产生不可预料的结果；(已淘汰)

问题：一，当线程终止时，很少有机会执行清理工作；二，当在某个线程上调用 stop()方法时，线程释放它当前持有的所有锁，持有这些锁必定有某种合适的理由——也许是阻止其他线程访问尚未处于一致性状态的数据，突然释放锁可能使某些对象中的数据处于不一致状态，而且不会出现数据可能崩溃的任何警告

- 使用interrupt方法中断线程；（推荐）

##  线程阻塞 yield()，sleep()
> sleep(ms)

暂停一段时间
>  yield()

yield()方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU执行时间。但放弃时间不确定，有可能刚刚放弃，马上又获得CPU时间片。这里需要注意的是**yield()方法和sleep方法一样，线程并不会让出锁**，和wait不同。


## DaemonThread守护线程
只要当前JVM实例中尚存任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束是，守护线程随着JVM一同结束工作，Daemon作用是为其他线程提供便利服务，守护线程最典型的应用就是GC(垃圾回收器)

- 设置守护线程必须在用户线程start前设置
- 在Daemon线程中产生的新线程也是Daemon
- java的线程池会将守护线程转换为用户线程，所以如果要使用后台线程就不能用java的线程池。
如下，线程池中将daemon线程转换为用户线程的程序片段
> 示例1

一个主线程(Daemon),一个用户线程，主线程结束，用户线程也结束

```
public class DaemonDemo1 {
    public static void main(String args[]) {
        Thread daemonThread = new Thread(new Runnable() {
            @Override
            public void run() {
                Thread t = new Thread(new UserThread());
                t.setDaemon(true);
                t.start();
                try {
                    TimeUnit.MILLISECONDS.sleep(1200);//
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("DaemonThread");
            }
        });
        daemonThread.start();
    }
}
class UserThread extends  Thread{
    public void run(){
        while(true){
            System.out.println("用户线程...");
            try {
                TimeUnit.MILLISECONDS.sleep(1000);//如果参数小于0，则不sleep
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
- 注：若当前jvm应用实例中有其他用户线程继续执行，那么后台线程不会中断
> 示例

```
public class DaemonDemo1 {
    public static void main(String args[]) {
        Thread daemonThread = new Thread(new Runnable() {
            @Override
            public void run() {
                Thread t = new Thread(new UserThread());
                t.setDaemon(true);
                t.start();
                try {
                    TimeUnit.MILLISECONDS.sleep(1200);//
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("DaemonThread");
            }
        });
        daemonThread.start();
        Thread daemonThread2 = new Thread(new Runnable() {
            public void run(){
                while(true){
                    try {
                        TimeUnit.MILLISECONDS.sleep(1200);//
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("DaemonThread2");
                }
            }
        });
        daemonThread2.start();
    }
}

class UserThread extends  Thread{
    public void run(){
        while(true){
            System.out.println("用户线程...");
            try {
                TimeUnit.MILLISECONDS.sleep(1000);//如果参数小于0，则不sleep
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```
用户线程...
用户线程...
DaemonThread
DaemonThread2
用户线程...
DaemonThread2
用户线程...
DaemonThread2 (无限执行)
```

