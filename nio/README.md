# NIO 教程目录

##### 这部分基本上是参考文末的参考资料，以及对照源码进行学习。大部分内容为直接翻译(也参考了并发编程网的翻译内容)，另外部分地方加入了些自己觉得能方便大家去理解和学习的内容。

Java NIO(New IO)是一个可以替代标准Java IO API和java Networking的IO API（从Java 1.4开始)，Java NIO提供了与标准IO不同的IO工作方式。

### Channels and Buffers（通道和缓冲区）

标准的IO下你是基于字节流和字符流进行操作的，而在NIO下是基于通道（Channel）和缓冲区（Buffer）进行操作的，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。

### Non-blocking IO（非阻塞IO）

Java NIO可以让你非阻塞的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。一旦数据被写入到缓冲区，线程便可以继续处理它。从缓冲区写入通道也类似。

### Selectors（选择器）

Java NIO引入了选择器的概念，选择器是一个用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

以下为本部分教程目录

### 1. [Overview](./overview.md)
### 2. [Channel](./channel.md)
### 3. [Buffer](./buffer.md)
### 4.	[Scatter|Gather](./scatter&gather.md)
### 5.	[Channel to Channel Transfers](./channel2channel.md)
### 6.	[Selector](./selector.md)
### 7.	[FileChannel](./filechannel.md)
### 8.	[SocketChannel](./socketchannel.md)
### 9.	[ServerSocketChannel](./serversocketchannel.md)
### 10. [Non-blocking Server](./non-blocking_server.md)
### 11.	[DatagramChannel](./datagramchannel.md)
### 12.	[Pipe](./pipe.md)
### 13.	[NIO VS IO](./nio_vs_io.md)
### 14.	[Path](./path.md)
### 15.	[Files](./files.md)
### 16.	[AsynchronousFileChannel](./asynchronousfilechannel.md)





参考资料:<http://tutorials.jenkov.com/java-nio/index.html>
