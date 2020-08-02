---
title: 锁的优化-ThreadLocal
date: 2019-09-17 12:00:00
tags:
  - 实战Java高并发程序设计
  - ThreadLocal
---

ThreadLocal是一个线程的局部变量，只有当前线程可以访问，是线程安全的。

<!--more-->

ThreadLocal人手一只笔，指的是

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class Test {
    static ThreadLocal<SimpleDateFormat> tl = new ThreadLocal<>();
    public static class ParseDate implements  Runnable{
        int i = 0;
        public ParseDate(int i){this.i = i;}

        @Override
        public void run() {
            try {
                if(tl.get() == null){
                    tl.set(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
                    Date t = tl.get().parse("2019-9-17 8:20"+i%60);
                    System.out.println(i+":"+t);
                }
            }catch (ParseException e){
                e.printStackTrace();
            }
        }
    }
}
```

下面看一下ThreadLocal的set方法和get方法

```java
public void set(T value){
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if(map != null){
		map.set(this,value);
	}else{
		createMap(t,value);
	}
}
```

```java
public T get(){
	Thread t = Thread.currentThread();
	ThreadMap map  = getMap(t);
	if(map != null){
		ThreadLocalMap.Entry e = map.getEntry(this);
		if(e != null)
		return (T)e.value;
	}
	return setInitialValue();
}
```

ThreadLocal的原理实际上是所有线程构成一张Map，Key为Thread.currentThread每个线程，value为T。每人一支笔即为每个线程一个T。

那这里就存在了一个问题，ThreadLocal存在于每个线程内，只要线程不结束，该线程的ThreadLocal将一直存在。

如果设置了固定大小的线程池，线程池中的线程会一直活跃，ThreadLocal也会一直存在占据内存，如果想要移除要使用ThreadLocal.remove()方法。

另外一个回收方法是将其设置为null，即ThreadLocal tl = null。

ThreadLocalMap在ThreadLocal的内部。

因为ThreadLocalMap对ThreadLocal实例是弱引用，而tl变量对ThreadLocal实例是强引用。所以将tl置为null后，失去tl的强引用后可能会导致回收。