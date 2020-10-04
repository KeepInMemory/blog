---
title: redis分布式锁
date: 2020-09-23 18:30:25
tags:
  - redis
---

redis分布式锁可以通过setnx （set if not extist）来实现

首先考虑的实现是在redis业务代码前加setnx设置一把锁，执行完业务代码后释放掉

对应的代码是

```java
Boolean result = redisTemplate.opsForValue().setIfAbsent("lockKey","lockValue");
//业务代码
redisTemplate.delete("lockKey");
```

在分布式的情况下，客户请求通过nginx分发到不同的服务器上，不同服务器最终会请求一个Redis服务器，redis是单线程的，各服务器的请求会排队进行处理，确保了隔离性

通过判断result获取锁的情况可以判断是否继续执行

**问题：如果在执行业务代码的时候抛异常了，那么delete不会得到执行**



于是考虑将加锁和解锁操作放在try finally里，确保能够释放锁

```java
try{
	Boolean result = redisTemplate.opsForValue().setIfAbsent("lockKey","lockValue");
	//业务代码
}
finally{
	redisTemplate.delete("lockKey");
}
```

**问题：如果在加锁后机器宕机了，那么这把锁永远不会释放掉**



考虑给setnx设置一个过期时间

```java
try{
	Boolean result = redisTemplate.opsForValue().setIfAbsent("lockKey","lockValue");
	redisTemplate.expire("lockKey",10,TimeUnit.SECONDS);
	//业务代码
}
finally{
	redisTemplate.delete("lockKey");
}
```

设置了10秒的过期时间，如果机器宕机了只能有10秒的空档期

**问题：如果A服务器执行了15秒才能执行完业务逻辑，在10秒的时候锁会自动过期，B服务器加锁成功，B正在执行，A执行完毕后把锁删除掉了，C服务器加锁成功，B执行完毕把锁释放掉了。这样导致锁和没设置一样**



考虑只能由自己删除掉自己的锁

给锁设置的value用线程自己的唯一标识，finally里判断一下，如果不是自己的锁就不删除

```java
String clientId = UUID.randomUUID().toString();
try{
	Boolean result = redisTemplate.opsForValue().setIfAbsent("lockKey","lockValue");
	redisTemplate.expire("lockKey",10,TimeUnit.SECONDS);
	//业务代码
}
finally{
	if(clientId.equals(redisTemplate.opsForValue().get("lockKey"))) {
		redisTemplate.delete("lockKey");
	}
}
```

**问题：当锁时间不够用，过期的时候仍然会有Bug，但是不严重了**



考虑给锁续命

线程在执行业务代码的时候，开启一个分线程执行一个定时任务，每隔一段时间（30/3=10s）检查一下这个线程加的锁是否还在，如果不再了说明自己解锁了，如果还在就给这把锁重新设置过期时间30s

这个就是redisson的底层实现，通过lua脚本，在Java方法里构造lua字符串发送给redis，redis会原子性的保证lua脚本被执行，中间出现了错误会回滚
