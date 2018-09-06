# 概述 Overview

- [NIO 简介](#nio-简介)
- [Channel 和 Buffer](#channel-和-buffer)
- [Selector](#selector)

### NIO 简介

>  java nio 包目录大概如下:
	
	nio
	├── channels	selector以及各种channel
	├── charset		
	├── file
	└── 各种buffer
  
其中核心部分为:

  - channel		位于channels包下
  - buffer		位于nio包下
  - seletor		位于channels包下

java NIO 除此之外还有很多的类和组件，但是在我看来channel、buffer、selector为java NIO API的核心。类似于pipe、fileLock 这样的其他组件只不过是连接这三个核心组件的工具类。因此，在概述中我们主要集中在这三个组件上，其他组件会在后续章节中讲到。

### Channel 和 Buffer

基本上，所有的 IO 在 NIO 中都是从一个 channel 开始。 channel 有点像流（但是流是单向的， channel 却是双向的），数据可以从 channel 读到 buffer ，也可从 buffer 写到 channel 。下面有张图示:

![](./pic/overview-channels-buffers.png)

channel 和 buffer 有好几种实现类型，下面 channel 在java NIO 中的一些主要实现:
- FileChannel
- DatagramChannel
- SockentChannel
- ServerSocketChannel

如你所见，这部分 channel 涵盖了 UDP 及 TCP 网络IO，和文件IO。
与这些实现类一起的还有些有趣的接口，为了让概述看起来简明，这里将不对这部分做更多的解释，而是在后续的其他部分中进行介绍。

下面是 buffer 在java NIO 中的一些主要实现:
- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
 
 这些 buffer 涵盖了你可以通过 IO 传输的所有基本数据类型:  byte, short, int, long, float, double and characters.
 java NIO 中还有一个 _MappedByteBuffer_ 用来连接映射文件，这种类型的 buffer 也不打算在概述中做说明。
 
 ### Selector
 
 一个 selector 允许单个线程去处理多个 channel 。这能很好的处理了当你的应用打开多个连接(channel) ，但每个连接(channel) 的流量都很低的情况。例如一个聊天服务器。
 
这有一张在单个线程中使用一个 selector 处理三个 channel 的示意图:

![](./pic/overview-selectors.png)

在同一个 selector 上，你可注册多个 channel ,然后你可以调用他的 select() 方法，这个方法将会阻塞，知道你注册的某个 channel 有事件准备就绪，一旦方法又返回，这个线程便可处理这些事件。关于事件的例子有:新连接的进来，数据接收等...


参考：<http://tutorials.jenkov.com/java-nio/overview.html>
