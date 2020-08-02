---
title: Java8的并发
date: 2019-09-22 12:00:00
tags:
  - 实战Java高并发程序设计
  - Java8
---

### 增强的Future：CompletableFuture

#### 没准备好就等待

向CompletableFuture请求一个数据，如果数据还没准备好，请求线程就会等待。也可以通过手动设置CompletableFuture的完成状态

```java
import java.util.concurrent.CompletableFuture;

public class AskThread implements Runnable{
    CompletableFuture<Integer> re = null;
    public AskThread(CompletableFuture<Integer> re){
        this.re = re;
    }
    @Override
    public void run() {
        int myRe = 0;
        try {
            myRe = re.get() * re.get();
        }catch (Exception e){
            System.out.println(myRe);
        }
    }

    public static void main(String[] args)throws InterruptedException{
        final CompletableFuture<Integer> future = new CompletableFuture<>();
        new Thread(new AskThread(future)).start();

        Thread.sleep(1000);
        future.complete(60);
    }
}
```

线程的作用是计算CompletableFuture表示的数字的平方。程序执行到

myRe = re.get() * re.get();时会阻塞，因为CompletableFuture没有需要的数据。最后一行diamagnetic表示CompletableFuture已经完成，因此线程可以继续运行并结束。

#### 异步执行任务

```java
public static Integer calc(Integer para){
	try{
		Thread.sleep(1000);
	}catch(InterruptedException e){}
	return para * para;
}
public static void main(String[] args)throws InterruptedException,ExecutionException{
    final CompletableFuture<Integer> future = CompletableFuture.supplyAsync(()->calc(50));
    System.out.println(future.get)();
}
```

通过使用CompletableFuture.supplyAsync()构造一个CompletableFuture实例，supplyAsync会在一个新线程中执行传入的参数。这里会让新线程执行calc函数，异步体现在了执行calc后立即返回，并不会等待参数执行完。返回后执行future.get方法，因为future的数据还没准备好，所以会等待。

```java
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,Executor executor);
static CompletableFuture<Void> runAsync(Runnable runnable);
static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor);
```

supplyAsync()用于需要返回值的场景，例如计算某个数据。runAsync()用于没有返回值的场景，例如简单地执行某个异步动作。

每个方法都可接收Executor，让Supplier或者Runnable在指定的线程池中工作，否则默认系统公共的线程池ForkJoinPool.common

#### 异常处理

CompletableFuture提供了一个异常处理方法exceptionally()

```java
public static Integer calc(Integer para){
    return para/0;
}
public static void main(String[] args)throws InterruptedException,ExecutionException{
    CompletableFuture<Void> fu = CompletableFuture
        .supplyAsync(()->calc(50));
    .exceptionally(ex->{
        System.out.println(ex.toString());
        return 0;
    })
     fu.get();
}
```

### 改进的读写锁：StampedLock

普通的读写锁采用悲观的锁策略，读和写之间是互斥的。而StampedLock则是乐观的读策略，类似于无锁的操作，使得读操作不会阻塞写操作。

```java
import sun.nio.cs.ext.MacArabic;

import java.util.concurrent.locks.StampedLock;

public class Point {
    private double x,y;
    private final StampedLock s1 = new StampedLock();

    void move(double x,double y){
        long stamp = s1.writeLock();
        try {
            this.x += x;
            this.y += y;
        }finally {
            s1.unlockWrite(stamp);
        }
    }

    double distanceFromOrigin(){
        long stamp = s1.tryOptimisticRead();
        double currentX = x,currentY = y;
        if(!s1.validate(stamp)){
            stamp = s1.readLock();
            try {
                currentX = x;
                currentY = y;
            }finally {
                s1.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX*currentX+currentY*currentY);
    }
}
```

每一次写操作都会修改时间戳

tryOptimisticRead乐观读方法，认为没有写线程在写，不管在没在写直接读出来，返回一个时间戳。此时要用validate判断时间戳在读过程中是否被修改过，如果没有修改过则认为这次读取是有效的。如果读取有效，直接执行最后一行Math.sqrt。如果读取无效，有两个选择，一种是选择把乐观锁变成悲观锁，本例中就是变成悲观锁后再读，此时读和写是互斥的，最后不管怎样都会执行读锁的解锁；另一种方法是重复乐观读，直到读取有效。

### 发布和订阅模式

Java9引入的并发编程架构反应式编程，反应式编程用于处理异步流的数据，反应式编程以流的形式处理数据，内存使用效率更高。

反应式编程的核心组件是Publisher和Subscriber。Publisher将数据发布到流中，Subscriber负责处理这些数据。

```java
//Subscriber是订阅者
onSubscribe():订阅者注册后被调用的第一个方法
onNext():当有下一个数据项准备好时，进行通知
onError():当发生无法恢复的异常时被调用
onComplete()：当没有更多数据需要处理时被调用

//Subscription是对订阅数据的处理
request()：设定请求的数据
cancel()：Subscribe停止接收新的消息
```