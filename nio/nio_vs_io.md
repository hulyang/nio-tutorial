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

![]()


### 总结

参考：<http://tutorials.jenkov.com/java-nio/nio-vs-io.html>
