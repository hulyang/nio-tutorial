# Channel To Channel Transfers

目录
- [transfer 简介](#transfer-简介)
- [transferFrom()](#transferfrom)
- [transferTo()](#transferto)

### transfer 简介

在java NIO 中，如果有一个channel为FileChannel，你可以直接某个channel中的数据转移到另外一个channel。FileChannel类中有transferFrom() 方法和 transferTo() 方法提供了这方面的支持。

> 这两个方法在均在实现类(FileChannelImpl)中实现，看不到具体实现过程，故这里不能为大家贴上源码了！

### transferFrom()

FileChannel.transferFrom() 方法将资源channel中的数据传输到FileChannel中。以下为代码示例：

```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();

toChannel.transferFrom(fromChannel, position, count);
```

这里的参数position和count，position是指目标文件(toFile)从哪开始写(写的position)，count是指最多一共传输多少字节。如果数据源(fromFile)数据量比较少，那就有多少传多少。

### transferTo()

FileChannel.transferTo() 方法将FileChannel中的数据传输到其他channel中。以下为代码示例：

```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

fromChannel.transferTo(position. count, toChannel);
```

看得出来这个例子和前面那个例子很是相似。唯一区别就是调用的方法不同而已，其他都是一致的、

SocketChannel中存在的问题，transferTo()方法也同样存在。SocketChannel在实现数据传输时，当buffer被填满后传输将会停止。
> 教程中这里出现SocketChannel个人感觉不是很合适，这里可以先不要去管SocketChannel，更不用去理解SocketChannel中有啥问题。我们只需要理解调用transferTo()时，如果目标被装满了(toChannel)，那么数据传输就会停止，即使还有数据没传输完。

参考：<http://tutorials.jenkov.com/java-nio/channel-to-channel-transfers.html>
