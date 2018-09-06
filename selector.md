# Selector

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

如[注册Channel到Selector](#注册channel到selector)中所述，interest set 是你选择监听的事件集。你可以像下面这样通过SelectionKey去读写这个事件集：

```
int interestSet = selectionKeys.interestOps();

boolean isInterestAccept = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT;
boolean isInterestConnect = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_CONNECT;
boolean isInterestRead = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_READ;
boolean isInterestWrite = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_WRITE;
```

> 原文中示例代码有点问题，我做了点改正。

如你所见，你可以用"&"(按位与)的方式去处理interest set和SelectionKey中的事件常量，去判断是否监听了某一事件。

##### Ready Set

ready  set 是已经准备就绪的事件集。在一次Selection你可能会首先访问这个ready set。Selection将在后面的章节中做具体解释，你可以像下面这样访问ready set：

```
int readySet = selectionKey.readyOps();
```

你可以同上面interest set中一样的方式来查看哪些事件已经准备就绪。但是，你也可以用以下4个方法来代替，他们都是返回boolean值：

```
selectionKey.isAcceptable();    // return (readyOps() & OP_ACCEPT) != 0
selectionKey.isConnectable();   // return (readyOps() & OP_CONNECT) != 0;
selectionKey.isReadable();      // return (readyOps() & OP_READ) != 0;
selectionKey.isWritable();      // return (readyOps() & OP_WRITE) != 0;
```

> 我在方法后面加了注释为方法的实现，其实跟我们上面做的差不多。

##### Channel + Selector

从SelectionKey中访问Channel和Selector是很频繁的，你可以通过一下方式实现：

```
Channel channel = selectionKey.channel();

Selector selector = selectionKey.selector();
```

##### Attaching Objects

你可以将一个对象attach给selectionKey，这样你能更好地识别某个特定的channel，又或者附加给channel更多的信息。例如你可以将与channel一同使用的buffer，或是包含更多聚合信息的对象attach上去。一下是如何attach对象的示例：

```
selectionKey.attach(theObject);
Object attachObj = selectionKey.attachment();
```

你也可以在注册channel到selector时，通过register()方法就将对象attach上去，下面是示例代码：

```
SelectionKey selectionKey = channel.register(selector, SelecitonKey.OP_READ, theObject);
```

### 通过Selector选择Channel

一旦你将一个或多个channel注册到同一个selector上以后，你可以调用某一select()方法，这些方法将返回有监听事件(accept、connect、read、write)准备就绪的channel。换句话说，如果你监听的channel中有read事件准备就绪，你就会通过调用select()方法返回一个read准备就绪的channel。

> 这里有点问题，select() 方法返回值为int，返回的是有时间准备就绪的channel的数量...并不是直接放回channel。

以下是几个select()方法：

```
int select();
int select(long timeout);
int selectNow();
```

select() 方法将会阻塞你注册的channel中有监听的事件准备就绪。
select(long timeout)和select()方法一样会阻塞，但是只是阻塞到你通过参数设置的超时时间(毫秒)的最大值为止。
selectNow() 不会阻塞，无论channel准备就绪与否都会立即返回。

select()方法返回的int值告诉我们有多少个channel已经准备就绪。这仅仅是返回在你上次调用select()后有多少channel准备就绪。如果你调用select()方法，并且返回值为1，说明有一个channel准备就绪，然后你在调用一次select()方法，并且又有一个channel准备就绪了，这次返回值还是1。如果你对第一个准备就绪的channel什么事都没做的话，你就有2个已经准备就绪的channel，但是只有在两次调用select()之间只有一个channel准备就绪。

##### selectedKeys()

一旦你调用了某个select()方法，并有正确的返回值，那就说明有channel已经准备就绪。你可以通过调用selectedKeys()方法返回的"select key set"来访问这些准备就绪的channel，以下为示例：

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();
```

当你通过channel.register()方法将channel注册到selector后，会返回一个SelectionKey，这个key记录着这个channel注册到那个selector。你可调用selectedKeys()方法来访问这些key。

你可迭代这一selected key set来访问准备就绪的channel，以下为代码示例：

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();

Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {
    
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
}
```

循环迭代selected key set中的key，然后通过key去判断其应用的channel是否有事件准备就绪。

注意每次循环最后的keyIterator.remove()，Selector是不会将selectionKey从selected key set中移除的。你得自己在处理完那个channel后来自己完成这件事。当下次有channel准备就绪，Selector将会再次将他加入到selected key set中去。

通过SelectionKey.channel()方法返回的channel，你需要自己去将他转型成你指定的那个类型，如ServerSocketChannel、SocketChannel等。

### wakeUp()

一个线程调用了select()方法导致线程阻塞，即使没有channel准备就绪，你也想马上退出select()方法。这时就需要在另外一个线程中使用之前调用select()方法的那个selector去调用Selector.wakeUp()方法，阻塞的select()方法将会立即返回。

如果某个线程调用了wakeUp()方法，但是当时没有线程阻塞在select()中，以后有线程再去调用select()方法将会立即返回。

### close()

如果你对selector的处理已经完成，你可以调用他的close()方法来关闭它。这个方法将关闭selector以及释放注册到selector中的所有selectionKey，但是注册的channel并不会关闭。

### 完整示例

以下是一个打开selector、为其注册channel(channel的初始化省略)、并持续监控selector中4个事件(accept,connect,read,write)是否准备就绪的完整示例：

```
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.select();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```

参考：<http://tutorials.jenkov.com/java-nio/selectors.html>
