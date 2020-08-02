---
title: JDK并发包-线程池
date: 2019-09-14 12:00:00
tags:
  - 实战Java高并发程序设计
  - 线程池
---

### JDK对线程池的支持

#### 固定大小线程池

JDK提供了Executor框架，本质就是个线程池

newFixedThreadPool方法：返回一个固定线程数量的线程池，多余的任务会被保存在任务队列中，等待空闲线程。

newSingleThreadExecutor方法：返回只有一个线程的线程池，多余的任务会被保存在任务队列中，等待空闲线程。

newCachedThreadPool方法：返回一个可根据实际情况调整线程数量的线程池。若线程池有空闲线程，则会有限使用空闲线程。如果所有线程均在工作又有新任务提交，会创建新的线程处理，完毕后放回线程池。

newSingleThreadScheduledExecutor方法：返回一个ScheduledExecutorService对象，线程池大小为1。ScheduledExecutorService接口在ExecutorService接口上增加了在一定时间执行某任务的功能。

newScheduledThreadPool方法：返回一个ScheduledExecutorService对象，线程池可以指定线程数量。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolDemo {
    public static class MyTask implements Runnable {
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + "Thread ID:" + Thread.currentThread().getId());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
        public static void main(String[] args){
            MyTask task = new MyTask();
            ExecutorService es = Executors.newFixedThreadPool(5);
            for(int i = 0; i < 10; ++ i){
                es.submit(task);
            }
        }
    }
```

程序输出

1568423198070Thread ID:13
1568423198070Thread ID:16
1568423198070Thread ID:15
1568423198070Thread ID:12
1568423198070Thread ID:14
1568423199071Thread ID:13
1568423199071Thread ID:16
1568423199072Thread ID:14
1568423199072Thread ID:12
1568423199072Thread ID:15

submit方法向线程池循环提交了10个任务，前五个任务在同一时间执行，后五个任务等前面的空出线程来再执行，相差一秒。

#### 计划任务

newScheduledThreadPool方法会返回一个ScheduledExectorService对象。

其中有三种方法

schedule方法会在给定时间对任务进行一次调度

scheduleAtFixedRate方法以上一个任务开始的时间为起点，间隔period时间进行一次调度

scheduleWithFixedDelay方法以上一个任务结束的时间为起点，间隔period时间进行一次调度

```java
import sun.security.krb5.internal.TGSRep;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledExecutorServiceDemo {
    public static void main(String[] args){
        ScheduledExecutorService ses = Executors.newScheduledThreadPool(10);
        ses.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    System.out.println(System.currentTimeMillis()/1000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        },0,2,TimeUnit.SECONDS);
    }
}
```

#### 核心线程池的内部实现

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

corePoolSize：指定了线程池中的线程数量，它的数量决定了添加的任务是开辟新的线程去执行，还是放到workQueue任务队列中去

maximumPoolSize：指定了线程池中的最大线程数量，这个参数会根据你使用的workQueue任务队列的类型，决定线程池会开辟的最大线程数量

keepAliveTime：当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁
unit:keepAliveTime的单位

workQueue：任务队列，被添加到线程池中，但尚未被执行的任务。它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列几种

threadFactory：线程工厂，用于创建线程，一般用默认即可

handler：拒绝策略，当任务太多来不及处理时，如何拒绝任务

#### 自定义线程池-拒绝策略

JDK内置的拒绝策略有

AbortPolicy：直接抛出异常，组织系统正常工作

CallerRunsPolicy：直接运行当前被丢弃的任务，当然任务的性能会急剧下降

DiscardOldestPolicy：丢弃最老的一个请求

DiscardPolicy：直接丢弃该任务

拒绝策略均需要实现RejectedExecutionHandler接口，自定义拒绝策略需要扩展该接口

```java
import java.util.concurrent.*;

public class RejectThreaPoolDemo {
    public static class MyTask implements Runnable{
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + "Thread ID :"+Thread.currentThread().getId());
            try {
                Thread.sleep(100);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args)throws InterruptedException{
        MyTask task = new MyTask();
        ExecutorService es = new ThreadPoolExecutor(5, 5, 0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(10), Executors.defaultThreadFactory(),
                new RejectedExecutionHandler() {
                    @Override
                    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                        System.out.println(r.toString()+"is discard");
                    }
                });
        for(int i = 0; i < Integer.MAX_VALUE; ++i){
            es.submit(task);
            Thread.sleep(10);
        }
    }
}
```

程序输出

1568426610713Thread ID :15
1568426610717Thread ID :16
java.util.concurrent.FutureTask@452b3a41is discard
java.util.concurrent.FutureTask@4a574795is discard

这里将常驻线程和最大线程数均设置为5，等待队列大小为10，因为无界等待队列可能会将内存撑爆，而自定义的拒绝策略将异常信息打印出来。

#### 自定义线程池-ThreadFactory

线程池中的初始线程通过ThreadFactory创建而来，ThreadFactory是一个接口，只有一个创建线程的方法。

```java
Thread newThread(Runnable r);
public static void main(String[] args)throws InterruptedException{
        MyTask task = new MyTask();
        ExecutorService es = new ThreadPoolExecutor(5, 5, 0L,
                TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>(),
                new ThreadFactory() {
                    @Override
                    public Thread newThread(Runnable r) {
                        Thread t = new Thread(r);
                        t.setDaemon(true);
                        System.out.println("create" + t);
                        return t;
                    }
                }
    );
```

本例自定义线程的创建，将所有线程设置为守护线程。

#### 扩展线程池

再默认的ThreadPoolExecutor实现中，提供了空的beforeExecute()和afterExecute()两个接口实现。实际应用中，对其进行扩展实现对线程状态的追踪，输出调试信息。

```java
public class ExtThreadPool {
    public static class MyTask implements Runnable {

        public String name;

        public MyTask(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println("正在执行" + Thread.currentThread().getId() + " Task" + name);
            try {
                Thread.sleep(100);
            }catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = new ThreadPoolExecutor(5,5,0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingDeque<>()) {

            @Override
            public void execute(Runnable command) {
                super.execute(command);
            }

            @Override
            protected void beforeExecute(Thread t, Runnable r) {
                System.out.println("准备执行" + ((MyTask)r).name);
            }

            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                System.out.println("执行完成" + ((MyTask)r).name);
            }

            @Override
            protected void terminated() {
                System.out.println("线程池退出");
            }
        };
        for (int i = 0; i < 5 ; i++) {
            MyTask myTask = new MyTask("Task"+i);
            executorService.execute(myTask);
            Thread.sleep(1000);
        }
        executorService.shutdown();
    }

}
```

最后的shutdown方法，关闭线程池，它不会暴力关闭，而是等到所有线程执行完毕后结束。

#### Fork/Join框架

Fork/Join框架采用分治的思想，将提交的大任务通过划分为小任务，分给各个线程。将各部分的结果结合起来成为最终结果。fork意为叉子，分叉。Join意为等待，等所有分任务都结束，合成结果。

Fork/Join框架有几个类，首先是ForkJoinTask类，该类表示任务类，支持fork方法和join方法。ForkJoinTask类有两个子类，RecursiveAction类和RecursiveTask类，前者表示没有返回值的任务，后者表示有返回值的任务。

下面是ForkJoinPool类，该类用来管理提交的任务以及fork出来的子线程。

```java
import java.util.ArrayList;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;

public class CountTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10000;
    private long start;
    private long end;

    public CountTask(long start,long end){
        this.start = start;
        this.end = end;
    }
    public Long compute(){
        long sum = 0;
        boolean canCompute = (end-start)<THRESHOLD;
        if (canCompute){
            for (long i = start;i <= end ;++i){
                sum += i;
            }
        }
        else{
            long step = (start + end)/100;
            ArrayList<CountTask> subTasks = new ArrayList<CountTask>();
            long pos = start;
            for(int i = 0 ; i < 100; ++ i){
                long lastOne = pos + step;
                if(lastOne > end)lastOne = end;
                CountTask subTask = new CountTask(pos,lastOne);
                pos += step+1;
                subTasks.add(subTask);
                subTask.fork();
            }
            for (CountTask t:subTasks){
                sum += t.join();
            }
        }
        return sum;
    }

    public static void main(String []args){
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        CountTask task = new CountTask(0,200000L);
        ForkJoinTask<Long> result = forkJoinPool.submit(task);
        try {
            long res = result.get();
            System.out.println("sum="+res);
        }catch (InterruptedException e){
            e.printStackTrace();
        }catch (ExecutionException e){
            e.printStackTrace();
        }
    }
}
```

本例中，CountTask类继承了RecursiveTask类，因为每个任务都是要有返回值的。通过submit把任务提交给ForkJoinPool。

线程池执行Task类的compute方法，将任务划分为若干子任务Task，将子任务放进ArrayList中，调用子任务Task的fork方法分治下去，最后将所有子任务Task调用其join方法，join方法会返回Task类的结果，可以通过Task类的get方法获取结果。