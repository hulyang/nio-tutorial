# Selector

目录
- [Selector 简介](#selector-简介)
- [为什么使用Selector](#为什么使用selector)
- [Selector的创建](#selector的创建)
- [注册Channel到Selector](#注册channel到selector)
- [SelectionKey](#selectionkey)
  - [Interest Set](#interest-set)
  - [Ready Set](#ready-set)
  - [Channel + Selector](#channel--selector)
  - [Attaching Objects](#attaching-objects)
- [通过Selector选择Channel](#通过selector选择channel)
  - [selectedKeys()](#selectedkeys)
- [wakeUp()](#wakeup)
- [close()](#close)
- [完整示例](#完整示例)

### Selector 简介

一个选择器(Selector)是java NIO 中的一个重要组件，其可监测多个Channel的状态，如谁已经准备读、谁已经准备写。这就是在单线程中去管理多个Channel的方式，因此可以同时管理多个网络连接。

### 为什么使用Selector

使用Selector的优势，就是在于你可以在单个线程中去管理多个Channel，这样你可用更少的线程去处理这些Channel。实际上，你完全可以只是用一个线程去处理所有的Channel。在多个线程之间来回切换对操作系统的开销是相当大的，并且每个线程都需要占用一定的操作系统的资源(内存)。因此启用的线程越少越好。

尽管如今的操作系统和CPU在多任务处理方面做得越来越好，多线程的开销也变得越来越小。事实上，如果一个多核CPU，你没有进行多任务都可能造成一定的能耗浪费。不管怎样，关于这种设计的讨论不属于本文范畴，在这里我们只要知道使用Selector可以让我们在单线程中操作多个Channel。

### Selector的创建

你可以通过Selector.open()来创建一个选择器，就像下面这样：
```
Selector selector = Selector.open();
```

### 注册Channel到Selector

为了将Channel和Selector一起使用，你就必须将Channel注册到Selector中，你可以通过SelectableChannel.register()方法来完成这一操作。就像下面这样：

```
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

> 这里提到的SelectableChannel是一个实现Channel的抽象类，他的常用的子类实现由我们之前提到过的DatagramChannel、SocketChannel、SeverSocketChannel。另外FileChannel是没有继承这样父类的，下面我贴出register()的具体实现(jdk1.8)，我会在代码中加入部分注释方便大家阅读。

```
public final SelectionKey register(Selector sel, int ops, Object att)
    throws ClosedChannelException
{
    synchronized (regLock) {
        if (!isOpen()) // 检查channel是否打开
            throw new ClosedChannelException();
        if ((ops & ~validOps()) != 0) // 检查操作参数是否合法
            throw new IllegalArgumentException();
        if (blocking) // 检查channel是否在阻塞模式下
            throw new IllegalBlockingModeException();
        SelectionKey k = findKey(sel); // 看看channel是否在已经注册了这个sel，有则返回已经注册好的key，没有则返回null
        if (k != null) {  // 有注册过，就加入新的关注事件和更新attach
            k.interestOps(ops);
            k.attach(att);
        }
        if (k == null) { // 没注册过，就通过sel.register()方法将本channel以及需要监听的事件注册进去。
            // New registration
            synchronized (keyLock) {
                if (!isOpen())
                    throw new ClosedChannelException();
                k = ((AbstractSelector)sel).register(this, ops, att);
                addKey(k); // 将key加入keys数组中，该方法会自动给keys扩容
            }
        }
        return k;
    }
}
```

与Selector一起使用的Channel必须在非阻塞的状态下，这就意味着你不能将FileChannel注册到Selector中使用，因为FileChannel不能切换到非阻塞的状态，而Socket Channel都是可以的。

注意register()的第二个参数，这是一个"interest set"，标志着你想通过Selector监听Channel的哪一事件。有以下4个事件供监听：

- Connect
- Accept
- Read
- Write

若一个Channel触发了某一事件，说明那个时间已经准备就绪。所以，如果一个Channel与其他服务已经连接成功，那就是Connect就绪，如果Channel接收到新来的连接就是Accept就绪，如果Channel中有数据待读取就是Read就绪，如果Channel需要被写入数据，那就是Write就绪。

这4个事件在SelectionKey中以一下4个常量表示：

- SelectionKey.OP_CONNECT
- SelectionKey.OP_ACCEPT
- SelectionKey.OP_READ
- SelectionKey.OP_WRITE

通过将事件常量用 "|"(按位或) 连接，你可以监听多个事件，示例代码如下：

```
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

### SelectionKey

在上一章节中明显能了解到，当你调用register()方法向Selector中注册Channel后，将返回给你一个SelectionKey对象，这个SelectionKey对象有以下几个属性值得我们关注一下：

- The interest set
- The ready set
- The Channel
- The Selector
- An attached object

我会在下面对这些属性一一讲解。

##### Interest Set



##### Ready Set

##### Channel + Selector

##### Attaching Objects

### 通过Selector选择Channel

##### selectedKeys()

### wakeUp()

### close()

### 完整示例
