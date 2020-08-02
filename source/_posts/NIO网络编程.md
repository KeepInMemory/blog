---
title: NIO网络编程
date: 2019-09-20 12:00:00
tags:
  - 实战Java高并发程序设计
  - NIO
---

使用Java的NIO可以将网络IO等待时间从业务处理线程中抽取出来。

NIO：准备好了再通知我

**Channel**：类似于流，可以和文件对应，往Channel中写就是往文件中写；可以和Socket对应，往Channel写就是往Socket中写

**Buffer**：一个内存区域，写入或者读取的数据需要包装成Buffer才能和Channel交互

**Selector**：Channel可以实习SelectableChannel类，表示可被选择的通道，实现这个类的Channel可以被Selector管理。一个Selector可以管理多个SelectableChannel。当SelectableChannel的数据准备好时，Selector会接到通知去处理数据。

<!--more-->

Socket Channel是SelectableChannel的一种，一个SoketChannel表示一个客户端连接。这些Socket  Channel由一个Selector管理，一个Selector由一个线程管理。所以可以通过一个或者少数线程来处理大量客户端连接。当客户端的数据没有准备好时，Selector通过轮询的方式等待。

```java
private Selector selector;
private ExecutorService tp = Executors.newCashedThreadPool();
```

Selector用于处理所有客户端连接，线程池用于对每一个客户端的请求进行处理。

```java
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.spi.SelectorProvider;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {
    private Selector selector;
    private ExecutorService tp = Executors.newCachedThreadPool();
    public static Map<Socket,Long> time_stat = new HashMap<>(10240);
    private void startServer() throws Exception{
        selector = SelectorProvider.provider().openSelector();
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);

        InetSocketAddress isa = new InetSocketAddress(InetAddress.getLocalHost(),8000);
        InetSocketAddress aisa = new InetSocketAddress(8000);
        ssc.socket().bind(isa);

        SelectionKey acceptKEy = ssc.register(selector,SelectionKey.OP_ACCEPT);

        for(;;){
            selector.select();
            Set readyKeys = selector.selectedKeys();
            Iterator i = readyKeys.iterator();
            long e = 0;
            while (i.hasNext()){
                SelectionKey sk = (SelectionKey)i.next();
                i.remove();

                if(sk.isAcceptable()){
                    //doAccept(sk);
                }
                else if(sk.isValid() && sk.isReadable()){
                    //doRead(sk);
                }
                else if(sk.isValid() && sk.isWritable()){
                    //doWrite(sk);
                }
            }
        }
    }
}
```