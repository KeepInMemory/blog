---
title: 锁的优化-建议与JVM
date: 2019-09-15 12:00:00
tags:
  - 实战Java高并发程序设计
  - 锁
---

### 提高锁性能的建议

#### 减少锁持有的时间

如果线程持有锁的时间越长，那么锁的竞争程度也就越激烈，等待的线程会越来越多。

```java
public synchronized void syncMethod{
	othercode1();
	mytexMethod();
	othercode2();
}
```

可以优化为

```java
public void syncMethod{
	othercode1();
	synchronized(this){
		mytexMethod();
	}
	othercode2();
}
```

只在必要时同步

#### 减小锁粒度

减小锁粒度，就是缩小指定对象的范围。典型的例子就是ConcurrentHashMap

对于HashMap来说，最重要的两个方法就是get()和put()。一种很自然的想法就是对整个HashMap加锁，这样肯定会得到一个线程安全的HashMap，但是这样做由于加锁粒度过大，会导致系统性能下降。

在ConcurrentHashMap内部进一步细分了若干个小的HashMap，称之为段(segment)，默认情况下ConcurrentHashMap内部被细分为16个段。

如果需要在ConcurrentHashMap中新增一个表项，并不需要将整个HashMap加锁，而是首先根据hashcode得到该表项应该被存放到哪个段，然后对该段加锁，并完成put()操作。在多线程环境中，如果多个线程同时进行put()操作，只要被加入的表项不存放在同一个段中，则线程间便可以做到真正的并行。

但这么做的缺点就是ConcurrentHashMap.size方法要获取这个信息需要取得所有子段的锁。

#### 锁分离

将独占锁的功能不同进行分离，比如分成读和写两部分。在读多写少的场合使用读写锁可以提高并发能力

#### 锁粗化

为此，虚拟机在遇到一连窜连续的对一个锁不断的进行请求和释放操作时，便会把所有的锁操作，整合成对锁的一次请求，从而减少对锁的请求同步次数，这个操作叫锁的粗化。

下面给出两个锁粗化的例子

```java
public void demoMethod(){
    synchronized(lock){
        //do sth.
    }
    //做其他不需要同步的工作，但很快能执行完毕
    synchronized(lock){
        //do sth.
    }
}
```

```java
public void demoMethod(){
    synchronized(lock){
        //do sth.
        //做其他不需要同步的工作，但很快能执行完毕
        //do sth.
    }
}

```

```java
for(int i = 0 ; i < k; ++ i){
	synchronized(lock){}
}
```

```java
synchronized(lock){
	for(int i = 0; i < k ; ++ i){
	
	}
}
```

### JVM的锁优化

#### 锁偏向

如果一个线程获得了锁，那么锁就进入偏向，当这个线程再次请求锁时，无需再做任何同步操作。这样节省了大量申请锁的时间。适用于几乎没有锁竞争的场合，在锁竞争激烈的场合还不如没有。

#### 锁消除

JVM在进行JIT编译时，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁。

例如，在没有并发的场合使用了Vector，锁消除使用的技术为逃逸分析，即分析一个变量是否会逃出某一个作用域。

```java
public String[] createStrings(){
	Vector<String> v= new Vector<String>();
    for(int i = 0 ; i < 100; ++ i){
        v.add(Integer.toString(i));
    }
    return v.toArray(new String[]{});
}
```

在本例种变量v没有掏出createStrings的作用域，所以可以将变量v内部的加锁操作去除。

如果返回的是变量v，则认为v有可能会被其他线程访问，不能消除加锁操作。

#### 轻量级锁

#### 自旋锁