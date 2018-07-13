# NIO VS IO

- [NIO VS IO 简介](#nio-vs-io-简介)
- [NIO与IO的主要区别](#nio与io的主要区别)
- [面向stream与面向buffer](#面向stream与面向buffer)
- [阻塞与非阻塞IO](#阻塞与非阻塞io)
- [选择器Selector](#选择器selector)
- [NIO与IO对程序设计的影响](#nio与io对程序设计的影响)
  - [API的调用](#api的调用)
  - [数据处理](#数据处理)
- [总结](#总结)

### NIO VS IO 简介

当学习了JAVA NIO和IO 之后，有个问题值得我们思考：

什么时候使用传统IO？什么时候使用NIO？

在本文中，我会尽力揭露NIO于IO的差异、NIO于IO的使用以及他们如何影响程序设计方面。

### NIO与IO的主要区别

下表总结了NIO与IO之间的主要区别。我会在后面章节中详细的阐述表中说提到的差异：

| IO | NIO |
| - | - |
| 面向流(stream) | 面向缓冲(buffer) |
| 阻塞IO | 非阻塞IO |
|  | 选择器Selector |

### 面向stream与面向buffer

JAVA NIO 与IO 最大的不同就是IO是面向流的，而NIO是面向缓冲的。这又意味着什么呢？

JAVA IO面向流，就意味着你从留中读取数据后，做什么操作取决你你自己，他不提供任何数据缓存。此外，你不可以在流中前后移动数据。如果你想在流中前后移动并从中读取数据，你的先将流中数据存入缓存才行。

JAVA NIO面向缓冲区则不同，数据会先读取到一个缓冲区(buffer)中，以待后续处理。你可以按照自己的需求在缓冲区中前后移动，这使得你在处理数据时更加灵活。然而，你也得去检查缓冲区中是否包含了你所需要处理的全量数据，以及必须确保读取更多数据到缓冲区时，不能覆盖缓冲区中还未来及处理的数据。

### 阻塞与非阻塞IO

JAVA IO 中各种流均是阻塞的。这就意味着，当某一线程调用流的read()或write()方法时，这一线程将会阻塞直到有数据从中读出，或是数据已经全部写入。这一线程在此时不能做其他任何事情。

JAVA NIO非阻塞模式下，使得一个线程向channel中读取数据时，会立即返回当前可获得的数据，如果当前没有可获得的数据，就啥都不返回，而不是阻塞到读取到有可获取的数据为止，这样这个线程就能继续做其他事了。

非阻塞模式下的写也是一样。一个线程需要向channel中写入数据时，不需要等待到数据完全被写入，这个线程同时就能继续做其他事。

线程通常可以将非阻塞IO的空闲时间，同时在其他channel上执行IO操作。这样一个线程就能同时管理多个channel的输入输出。

### 选择器Selector

JAVA NIO selector允许一个线程监控多个channel的输入。你可以将多个channel注册到同一selector，然后在单个线程中可以“选择”有可获得的输入数据的channel去处理，或是选择已经准备写入的channel。这种选择器机智单线程可以轻易的管理多个channel。

### NIO与IO对程序设计的影响

无论你选择NIO还是IO作为你的IO工具，都会影响你以下方面的程序设计。
1. 对NIO或IO类的API的调用
2. 数据处理
3. 处理数据的线程数

##### API的调用

当然，API的调用在使用NIO的时候和使用IO的时候是不一样的。这不奇怪，毕竟IO是从流中一个字节一个字节的读取数据，而NIO是将数据先读入buffer中，然后再从中处理数据。

##### 数据处理

当仅使用JAVA NIO 或者 JAVA IO时，数据处理过程上的设计也是会受到影响的。

在IO的设计中，你从inputstream或reader中读取数据，想象你正在处理一基于行的文本数据流，例如：

```
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```

文本的行的流可能处理如下：
```
InputStream input = ... ; // get the InputStream from the client socket

BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```

注意，这里处理状态有程序执行多久来决定。换句话说，一旦第一次执行的reader.readLine()返回了，你可以确定一整行被读取了，readLine()方法会阻塞直到整行都被读完，这就是原因。你已经知道了这行内容包含着名字，同样，当第二次调用readLine()并返回时，你知道这行内容包含着年级。

如你所见，这段程序处理仅当读取到新的数据时，并每次你都知道读取到的是什么数据。一旦执行线程已经将某个确定的数据处理过以后，这个线程是不能再回退到该部分数据上的(大部分如此)。下图是这一原理的示例：

![](./pic/nio-vs-io-1.png)

NIO的实现会有所不同，下面是一个简单的示例：

```
ByteBuffer buffer = ByteBuffer.allocate(48);
int byteRead = inChannel.read(buffer);
```

注意第二行，将数据从channel读取到buffer中。当那个方法调用并返回后，你并不知道是不是你所需的全部数据都已经在buffer中。你所知道的只是buffer中含有一些数据。这就导致数据处理有些困难。

想象以下，当第一次调用read(buffer)以后，此次调用读入buffer的只有半行，例如："Name: An"，你能处理这样的数据吗？显然不能。你得等到直到一整行数据都被读入buffer中，在此之前，对数据的任何处理都显得毫无意义。

所以，你如何才能知道buffer中已经包含能够处理的所有数据了呢？可惜，你不知道。唯一方法就是去查看buffer中的数据。这样做的结果就是在确定所有数据都存在buffer之前，你得多次检查buffer中的数据。这不仅效率低下，而且会使得程序看起来一片混乱。例如：

```
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);

while(! bufferFull(bytesRead) ) {
    bytesRead = inChannel.read(buffer);
}
```

bufferFull()方法会不断监测有多少数据被读入buffer，并返回true或false，以表示buffer是不是已经满了，换句话说就是buffer是不是已经准备好被处理了，那就认为buffer已经满了。

bufferFull()方法扫描整个buffer，并且必须保证buffer在bufferFull()方法调用结束时与调用前状态一致。如果不一致，那么下次读入的数据将不能保存在正确的位置上，这不是不可能的，而是必须注意的一个问题。

一旦buffer满了，就表示buffer已经可以被处理了。但如果buffer没满，在特殊情况下，你可能也需要处理其中部分数据。虽然大部分情况下不会这样。

以下是一个data->buffer->ready循环的示意图：

![](./pic/nio-vs-io-2.png)

### 总结

NIO允许你在单线程(或少量线程)中同时管理多个channel(网络连接或者文件)，但是代价就是解析数据可能回比从阻塞IO中读取到的数据难度要大一些。

如果你需要同时管理数以千计的打开的链接，并且这些链接都只传输少量数据，例如一个聊天服务器，通过NIO实现这项服务可能有一定优势。相似的，如果你与其他电脑之间得保持大量打开的的链接，例如一个P2P网络，使用单个线程去管理你所有的出站链接可能具有一定优势。单线程管理多链接的示意图如下：

![](./pic/nio-vs-io-3.png)

如果你只有几个少量的链接，但是得传输大量数据，占用很大的带宽，可能传统IO服务实现起来会更合适。以下是一个传统IO服务的示意图：

![](./pic/nio-vs-io-4.png)

参考：<http://tutorials.jenkov.com/java-nio/nio-vs-io.html>
