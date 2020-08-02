---
title: JDK并发包-同步控制
date: 2019-09-11 12:00:00
tags:
  - 实战Java高并发程序设计
  - 同步
---

### 重入锁

------

重入锁可以完全替代synchronized

```java
import java.util.concurrent.locks.ReentrantLock;

public class ReenterLock implements Runnable{
    public static ReentrantLock lock = new ReentrantLock();
    public static int i = 0;

    @Override
    public void run() {
        for(int j = 0 ; j < 10000000;j ++){
            lock.lock();
            try {
                i++;
            }finally {
                lock.unlock();
            }
        }
    }
    public static void main(String[] args)throws InterruptedException{
        ReenterLock tl = new ReenterLock();
        Thread t1 = new Thread(tl);
        Thread t2 = new Thread(tl);
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
```

另外重入锁可以让一个线程多次加锁，但加了多少锁最后要释放多少锁。

```java
lock.lock();
lock.lock();
try{
	i++;
}finally{
	lock.unlock();
	lock.unlock();
}
```

<!--more-->

#### 中断响应

------

lockInterruptibly方法允许线程在等待锁的时候被interrupt方法打断，释放自己所拿到的锁，便于解除死锁。

isHeldByCurrentThread方法用于查询当前线程是否持有此锁，返回值boolean

```java
import java.util.concurrent.locks.ReentrantLock;

public class IntLock implements  Runnable {
    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();
    int lock;
    public IntLock(int lock){
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            if(lock == 1){
                lock1.lockInterruptibly();
                try {
                    Thread.sleep(500);
                }catch (InterruptedException e){}
                lock2.lockInterruptibly();
            }
            else{
                lock2.lockInterruptibly();
                try {
                    Thread.sleep(500);
                }catch (InterruptedException e){}
                lock1.lockInterruptibly();
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }finally {
            if(lock1.isHeldByCurrentThread())
                lock1.unlock();
            if(lock2.isHeldByCurrentThread())
                lock2.unlock();
            System.out.println(Thread.currentThread().getId()+"线程退出");
        }
    }
    public static void main(String[]args)throws InterruptedException{
        IntLock r1 = new IntLock(1);
        IntLock r2 = new IntLock(2);
        Thread t1 = new Thread();
        Thread t2 = new Thread();
        t1.start();t2.start();
        Thread.sleep(1000);
        t2.interrupt();
    }
}
```

主进程先开始执行，创建了t1和t2两个子线程后自己开始睡觉，睡10秒。

子线程t1开始执行，判断自己的lock=1后申请一号锁，申请到后自己睡5秒。

子线程t2开始执行，判断自己的lock=2后申请二号锁，申请到后自己睡5秒。

本来该主线程执行的，但是大家都在睡觉，但是t1是先醒的所以t1先执行。

t1申请二号锁，申请不到，在等待。

t2申请一号锁，申请不到，在等待。

主线程睡醒，中断了t2。t2释放自己所获得的二号锁，t1获得锁后结束。t2被中断了也结束了。

**锁申请等待限时**

tryLock方法用于申请锁，构造参数为等待时间，其中的5代表5个单位时间，后面的TimeUnit.SECONDS代表单位时间是秒。

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class TimeLock implements Runnable {
    public static ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        try {
            if(lock.tryLock(5,TimeUnit.SECONDS)){
                Thread.sleep(600);
            }
            else {
                System.out.println("get lock failed");
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        finally {
            if(lock.isHeldByCurrentThread())
                lock.unlock();
        }
    }
    public static void main(String[] args){
        TimeLock tl = new TimeLock();
        Thread t1 = new Thread(tl);
        Thread t2 = new Thread(tl);
        t1.start();t2.start();

    }
}
```

因为有两个线程在竞争锁，所以不知道哪个线程获得了锁，要使用isHeldByCurrentThread方法判断该子线程是否持有锁，持有的话在解除。

tryLock也有无参数的构造方法，用于申请锁，如果申请到则返回true，没申请到则直接返回不等待，返回false。线程不会等待锁，则避免了死锁。

```java
import java.util.concurrent.locks.ReentrantLock;

public class TryLock implements Runnable{
    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();
    int lock;

    public TryLock(int lock){
        this.lock = lock;
    }

    public void run(){
        if(lock == 1) {
            while (true) {
                if (lock1.tryLock()) {
                    try {
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) { }
                        if (lock2.tryLock()) {
                            try {
                                System.out.println(Thread.currentThread().getId() + ":My Job Done");
                                return;
                            } finally {
                                lock2.unlock();
                            }
                        }
                    } finally {
                        lock1.unlock();
                    }
                }
            }
        }
                else{
                    while(true){
                        if(lock2.tryLock()){
                            try {
                                try {
                                    Thread.sleep(500);
                                }catch (InterruptedException e){}
                                if(lock1.tryLock()){
                                    try {
                                        System.out.println(Thread.currentThread().getId()+":My Job Done");
                                        return;
                                    }finally {
                                        lock1.unlock();
                                    }
                                }
                            }finally {
                                lock2.unlock();
                            }
                        }
                    }
                }
            }
    public static void main(String[] args)throws InterruptedException{
        TryLock r1 = new TryLock(1);
        TryLock r2 = new TryLock(2);
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();t2.start();
    }
}
```

输出为

13:My Job Done
12:My Job Done

主线程创建了子线程t1和t2后结束

子线程t1开始执行，获取1号锁后睡0.5秒

t2线程开始执行，获取2号锁后睡0.5秒

t1线程使用tryLock获取2号锁，发现申请不到立即返回false，去执行下面的逻辑。t1线程释放了1号锁进入下一个循环。获取1号锁后睡0.5秒。

t2线程使用tryLock获取1号锁，申请不到返回false，释放2号锁后进入循环睡0.5秒。

以此循环不断重试，在系统的调度下，总会出现一个线程先完成的情况。因为后台的线程很多，不断的打断，不断的竞争cpu，当出现了t1释放了1号锁还没睡着前，被t2线程申请到了锁；或者t2释放了2号锁还没睡着前，被t1线程申请到了锁，死锁就会被破。

**公平锁**

大多情况下锁是非公平的，也就是说多个线程同时申请锁，系统会从锁的等待队列中随机挑选一个。

重入锁可以设置公平性

```java
public ReentrantLock(boolean fair)
```

当fair为true时，这个重入锁就是公平锁，公平锁保证了不会产生饥饿，但是因为要维护一个有序队列所以性能会变低。

```java
import java.util.concurrent.locks.ReentrantLock;

public class FairLock implements  Runnable{
    public static ReentrantLock fairLock = new ReentrantLock(true);

    @Override
    public void run() {
        while(true){
            try {
                fairLock.lock();
                System.out.println(Thread.currentThread().getName()+"获得锁");
            }finally {
                fairLock.unlock();
            }
        }
    }
    public static void main(String[] args)throws InterruptedException{
        FairLock r1 = new FairLock();
        Thread t1 = new Thread(r1,"Thread_t1");
        Thread t2 = new Thread(r1,"Thread_t2");
        t1.start();t2.start();
    }
}
```

程序输出是Thread_t1和Thread_t2交替获得锁。

若重入锁是非公平的，根据系统的调度，一个线程会倾向于再次获取已经持有的锁，这是较为高效的分配，但是却不公平。

**重入锁方法比较**

synchronized：获取锁，如果锁被占用，则等待直到释放锁

lock()：获取锁，如果锁被占用，则等待直到释放锁，不响应中断

unlock()：释放锁

lockInterruptibly()：获取锁，如果锁被占用，则等待直到释放锁，可以响应中断

tryLock()：尝试获取锁，成功返回true，失败返回false后继续执行不等待

tryLock(long time,TimeUnit unit)：在限定的时间内尝试获取锁，超时不等待

所有没有请求到锁的线程会进入等待队列中，释放CPU，待有线程释放锁后，系统会唤醒等待队列中的一个线程，去竞争CPU。

#### Condition

------

Condition的作用和wait()、notify()大致相同，但wait是和synchronized配合，Condition则是和重入锁配合。

await()方法会使当前线程等待，同时释放当前锁，可以被中断等待

awaitUniterruptibly())方法会使当前线程等待，同时释放当前锁，等待不能被中断

signal()从等待队列中唤醒一个在等待的线程

signalAll()唤醒所有在等待的线程

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ReenterLockCondition implements Runnable {
    public static ReentrantLock lock = new ReentrantLock();
    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
        try {
            lock.lock();
            condition.await();
            System.out.println("Thread is going on");
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    public static void main(String[] args)throws InterruptedException{
        ReenterLockCondition tl = new ReenterLockCondition();
        Thread t1 = new Thread(tl);
        t1.start();
        Thread.sleep(2000);
        lock.lock();
        condition.signal();
        lock.unlock();
    }
}
```

这里和wait、notify一样，在调用await的时候会释放掉持有的锁，但singal时会唤醒等待的线程参与竞争，之后线程在往下执行前会先获取之前的锁，获取成功了才能往下执行。

本例中关键的地方在最后的代码lock.unlock，只有主线程释放了等待线程之前的锁，唤醒后的它重新获取锁后才能继续执行。

#### 信号量

------

不管是内部锁synchronized还是重入锁ReentrantLock，只允许一个线程访问一个资源。

而信号量则可以指定多个线程，同时访问一个资源。

```java
public Semaphore(int permits)//构造信号量对象，设置准入数
public Semaphore(int permits,boolean fair)//第二个参数为是否公平

public void acquire()
public void acquireUniterruptibly()
public boolean tryAcquire()
public boolean tryAcquire(long time,TimeUnit unit)
public void release()
```

acquire方法获取一个准入许可，若无法获得会等待直至释放一个许可或者线程被中断

acquireUniterruptibly和acquire类似，但不响应中断

tryAcquire方法尝试获取一个许可，成功返回true，失败返回false，不等待继续执行

release方法释放一个许可

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemapDemo implements Runnable{
    final Semaphore semp = new Semaphore(5);

    @Override
    public void run() {
        try {
            semp.acquire();
            Thread.sleep(2000);
            System.out.println();
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            semp.release();
        }
    }
    public static void main(String[] args){
        ExecutorService exec = Executors.newFixedThreadPool(20);
        final SemapDemo demo = new SemapDemo();
        for (int i = 0 ; i < 20; ++ i){
            exec.submit(demo);
        }
    }
}
```

#### ReadWriteLock读写锁

------

使用内部锁或者重入锁实现读写，读操作之间是不能同时进行的，这不合理。使用读写锁，可以让多个进程同时读，但读与写，写与写之间仍然是互斥的。

```java
import java.util.Random;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;
import java.util.concurrent.locks.Lock;
public class ReadWriteLockDemo {
    private static Lock lock = new ReentrantLock();
    private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();//读写锁
    private static Lock readLock = readWriteLock.readLock();//读写锁里的读锁
    private static Lock writeLock = readWriteLock.writeLock();//读写锁里的写锁
    private int value;
    public Object handleRead(Lock lock)throws InterruptedException{
        try{
            lock.lock();
            Thread.sleep(1000);
            return value;
        }finally {
            lock.unlock();
        }
    }
    public void handleWrite(Lock lock,int index)throws InterruptedException{
        try {
            lock.lock();
            Thread.sleep(1000);
            value = index;
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args){
        final ReadWriteLockDemo demo = new ReadWriteLockDemo();
        Runnable readRunnable = new Runnable() {
            @Override
            public void run() {
                try {
                    demo.handleRead(readLock);
                    //demo.handleRead(lock);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        };
        Runnable writeRunnale = new Runnable(){
            @Override
            public void run() {
                try {
                    demo.handleWrite(writeLock,new Random().nextInt());
                    //demo.handleWrite(lock,new Random().nextInt());
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        };

        for (int i = 0 ; i < 18; ++ i){
            new Thread(readRunnable).start();
        }

        for (int i = 18; i< 20; ++i){
            new Thread(writeRunnale).start();
        }
    }
}
```

主线程被创建后开始执行，创建了18个读线程，2个写线程，主线程结束后读线程开始运行。

读线程运行readRunnable的run方法，首先调用自己写的读操作，将读锁传入。读锁锁住后线程睡1秒后返回解锁。因为demo.handleRead(readLock);使用的是读锁所以18个读线程是并行的。而2个写线程之间是互斥的。

这里读操作和写操作都要睡一秒，所以使用读写锁，程序总共需要花费2秒，这2秒是写进程睡觉的时间。

如果使用重入锁，程序总共需要20多秒才能结束，包含读进程睡觉的时间。

#### 倒计时器CountDownLatch

------

倒计时器用来控制线程等待，可以让某个线程等待直到倒计时结束，再执行。

```java
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchDemo implements Runnable {
    static final CountDownLatch  end = new CountDownLatch(10);
    static final CountDownLatchDemo demo = new CountDownLatchDemo();

    @Override
    public void run() {
        try {
            Thread.sleep(new Random().nextInt(10)*1000);
            System.out.println("check complete");
            end.countDown();
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
    public static void main(String[] args) throws  InterruptedException{
        ExecutorService exec = Executors.newFixedThreadPool(10);
        for(int i = 0 ; i < 10 ; ++ i){
            exec.submit(demo);
        }
        end.await();
        System.out.println("Fire");
        exec.shutdown();
    }
}
```

输出为

check complete
check complete
check complete
check complete
check complete
check complete
check complete
check complete
check complete
check complete
Fire

主线程创建包含10个线程的线程池，end.wait方法使当前线程等待所有检查任务完成，主线程再执行。倒计数器使得主线程等待10个子线程结束，自己才能继续执行。

每个子线程的run方法首先花费一定的时间去检查（睡觉），输出检查完成后使用end.countDown将计数减一。

#### 循环栅栏CyclicBarrier

------

循环栅栏和CountDownLatch功能类似，都是用于计数

```java
public CyclicBarrier(int parties,Runnable barrierAction)
```

第一个参数为要计的数目，第二个参数是实现了Runnable接口的类，重写run方法，当计数到0时会执行run方法并且重新计数。

await方法线程告诉CyclicBarrier自己已经到达同步点，然后当前线程被阻塞，直到parties个参与线程调用了await方法

```java
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static class Soldier implements Runnable {
        private String soldier;
        private final CyclicBarrier cyclic;

        public Soldier(String soldier, CyclicBarrier cyclic) {
            this.soldier = soldier;
            this.cyclic = cyclic;
        }

        @Override
        public void run() {
            try {
                //等待所有士兵到齐
                cyclic.await();
                doWork();
                //等待所有士兵去工作
                cyclic.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }

        private void doWork() {
            try {
                Thread.sleep(Math.abs(new Random().nextInt() % 10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(soldier + ":任务完成！");
        }
    }

    public static class BarrierRun implements Runnable {
        boolean flag;
        int N;

        public BarrierRun(boolean flag, int n) {
            super();
            this.flag = flag;
            N = n;
        }

        @Override
        public void run() {
            if (flag) {
                System.out.println("司令：【士兵" + N + "个，任务完成】");
            } else {
                System.out.println("司令：【士兵" + N + "个，集合完毕】");
                flag = true;
            }

        }
    }

    public static void main(String[] args) {
        final int N = 10;
        Thread[] allSoldier = new Thread[N];
        boolean flag = false;
        CyclicBarrier cyclic = new CyclicBarrier(N, new BarrierRun(flag, N));
        //设置障碍点，主要是为了执行这个方法
        System.out.println("集合队伍");
        for (int i = 0; i < N; ++i) {
            System.out.println("士兵" + i + "报道！");
            allSoldier[i] = new Thread(new Soldier("士兵" + i, cyclic));
            allSoldier[i].start();
        }
    }
}
```

主线程初始化了循环栅栏，并且把BarrierRun传递过去，创建了线程数组，将十个子线程初始化并且启动

子线程执行Soldier中的run方法，调用await方法后等待，直到十个子线程全部调用了await方法，十个线程都到齐后，计数到0后执行传给循环栅栏的BarrierRun类的run方法，放开栅栏，开始doWork，doWork方法让线程沉睡不同的时间，先完成的线程会再次调用wait方法等待，计数到0后再次执行传给循环栅栏的BarrierRun类的run方法

这里需要注意，BrokenBarrierException异常是循环栅栏特有的，如果给第五个士兵加入中断

```java
if(i == 5){
	allSoldier[0] .interupt();
}
```

这样做会得到1个InterruptedException和9个BrokenBarrierException，其中第五个士兵线程发出InterruptedException，其他线程发出BrokenBarrierException。处理BrokenBarrierException这个异常可以避免其他9个线程进行无谓的等待。

#### 线程阻塞工具LockSupport

------

LockSupport对比Thread.suspend()，弥补了resume方法在suspend前调用会无法唤醒的情况，和Object.wait相比，不需要事前获取某个对象的锁，也不会抛出InterruptedExcption

```java
import java.util.concurrent.locks.LockSupport;

public class LockSupportDemo {
    public static Object u = new Object();
    static ChangeObjectThread t1 = new ChangeObjectThread("t1");
    static ChangeObjectThread t2 = new ChangeObjectThread("t2");

    public static class ChangeObjectThread extends Thread{
        public ChangeObjectThread(String name){
            super.setName(name);
        }

        @Override
        public void run() {
            synchronized (u){
                System.out.println("in"+getName());
                LockSupport.park();
            }
        }
    }
    public static void main(String [] args)throws InterruptedException{
        t1.start();
        Thread.sleep(100);
        t2.start();
        LockSupport.unpark(t1);
        LockSupport.unpark(t2);
        t1.join();
        t2.join();
    }
}
```

这个例子在之前的suspend和resume中见过，在那里t1会执行完毕，但是t2会在输出后被挂起，但因为resume先发生了， 所以程序无法结束。

但LockSupport的底层原理类似于信号量机制，每个线程都有一个许可permit，permit只有两个值1和0，默认是0

当调用unpark(thread)方法，就会将thread线程的许可permit设置成1(注意多次调用unpark方法，不会累加，permit值还是1)。

当调用park()方法，如果当前线程的permit是1，那么将permit设置为0，**并立即返回**。如果当前线程的permit是0，那么当前**线程就会阻塞**，直到别的线程将当前线程的permit设置为1。park方法会将permit再次设置为0，并返回。

所以在这里子线程t1开始执行，获取对象u的锁后被挂起peimit=0，主线程睡好后创建t2，唤醒t1，t1的peimit=1。唤醒t2，设置t2的peimit=1。主线程等待t1和t2。

t1睡醒后将u的锁释放，线程结束。

t2获取u锁后挂起，peimit=0。因为之前的unpark把许可设为1，后面的park看到有许可，获取许可后就可立即返回，所以可以继续执行下去。

因为LockSupport.park()方法不会抛出InterruptedException异常，而是立即返回。所以可以从Thread.Interrupted判断是否被中断了。

#### Guava和RateLimiter限流

------

限流算法一般有两种：漏桶算法和令牌桶算法

RateLimiter使用令牌桶算法

```java
public class RateLimiterDemo {
    static RateLimter limter = RateLimter.create(2);

    public static class Task implements  Runnable{
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis());
        }
    }
    public static void main(String[] args)throws InterruptedException{
        for (int i = 0; i < 50; ++ i){
            limter.acquire();
            new Thread(new Task()).start();
        }
    }
}
```

本例的第二行规定了每秒往令牌桶中加入两个令牌，即规定每秒只能处理两个请求。

acquire方法会进行等待，如果想让无法处理的请求丢掉，可以使用tryAcquire尝试获取，如果获取失败则进入下一循环。