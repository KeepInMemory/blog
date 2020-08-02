---
title: 线程安全与synchronized
date: 2019-09-11 12:00:00
tags:
  - 实战Java高并发程序设计
  - sync
---

先看一个例子

```java
public class AccountingVol implements Runnable{
    static AccountingVol instance = new AccountingVol();
    static volatile int i = 0;
    public static void increase(){
        i++;
    }

    @Override
    public void run() {
        for(int j = 0 ; j < 10000000; j++){
            increase();
        }
    }
    public static void main(String[]args) throws InterruptedException{
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
```

这里虽然使用了volatile变量，但是线程是不安全的，也就是没有保证原子性。两个线程都执行2000000次i++但得到的结果是小于2000000的。原因在于如果t1读到数0，切换到t2读数0，t1和t2回写，结果是1。虽然i++被执行了两次但结果仍然是1。这里存在线程安全、线程同步的问题

synchronized的用法：

指定加锁对象：对给定对象加锁，进入同步代码前要获得给定对象的锁。

直接作用于实例方法：相当于对当前实例加锁，进入同步代码前要获得当前实例的锁。

直接作用于静态方法：相当于对当前类加锁，进入同步代码前要获得当前类的锁。

因此上面的例子可以用以下三种方式实现线程的安全

```java
public class AccountingVol implements Runnable{
    static AccountingVol instance = new AccountingVol();
    static volatile int i = 0;
    public static void increase(){
        i++;
    }

    @Override
    public void run() {
        for(int j = 0 ; j < 10000000; j++){
            synchronized (instance) {//进入代码块前必须获得当前对象实例的锁
                increase();
            }
        }
    }
    public static void main(String[]args) throws InterruptedException{
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}

```

```java
public class AccountingVol implements Runnable{
    static AccountingVol instance = new AccountingVol();
    static volatile int i = 0;
    public synchronized void increase(){//进入increase前必须获得当前对象实例的锁
        i++;
    }

    @Override
    public void run() {
        for(int j = 0 ; j < 10000000; j++){
                increase();
        }
    }
    public static void main(String[]args) throws InterruptedException{
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
```

但如果线程t1和t2的Runnable实例并不是同一个对象，那么上面两个方法的同步就是错误的，它们都需要同一个对象instance。所以如果对象不同，则可以通过sync+static解决。

```java
public class AccountingVol implements Runnable{
    static AccountingVol instance = new AccountingVol();
    static volatile int i = 0;
    public static synchronized void increase(){//进入方法前必须获得类的锁
        i++;
    }

    @Override
    public void run() {
        for(int j = 0 ; j < 10000000; j++){
                increase();
        }
    }
    public static void main(String[]args) throws InterruptedException{
        Thread t1 = new Thread(new AccountingVol());//两个线程没有指向同一个实例
        Thread t2 = new Thread(new AccountingVol());
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
```

### 线程安全需要注意的地方

------

在并发程序中使用ArrayList会出现上面的线程不安全例子，所以推荐使用Vector代替ArrayList，Vector是线程安全的。同样的，需要用ConcurrentHashMap代替HashMap

由于Integer对象的值一旦确定就不会改变，如果要改变则需要创建新的Integer。若要在增或减的Integer对象上加锁，会出现锁加在了不同的对象上的情况。