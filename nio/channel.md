# 通道 channel

通道(channel) 类似流，但是又有些许不同:
- 一个 channel 既可读，亦可写，但是流都是单向的(只读|只写)
- 通道可以异步读写
- 通道总是读写到 buffer 

如上所述，你可将channel 中的数据读取到 buffer, 也可将 buffer 中的数据 写入到 channel , 下面是张很好的示例图:

![](./pic/overview-channels-buffers.png)

### channel 的实现类

以下是 channel 在java NIO 中的最主要的几个实现类:
- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

FileChannel 可用来对文件进行读写操作
DatagramChannel 可通过 UDP 连接读写网络中数据
SocketChannel 可通过 TCP 连接读写网络中数据
ServerSocketChannel 可用像 web 服务那样，监听新进来的 TCP 连接，每当有新的 TCP 连接进来，就会为其创建一个新的 socketChannel 进行通信。

### channel 的基本示例

以下是一个使用 FileChannel 将数据读取 到 Buffer 的简单示例:
```
  RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
  FileChannel inChannel = aFile.getChannel(); // 获取Filechannel

  ByteBuffer buf = ByteBuffer.allocate(48); // 创建一个ByteBuffer

  int bytesRead = inChannel.read(buf);  // 从channel中读取数据到buffer
  while (bytesRead != -1) {
    System.out.println("Read " + bytesRead);
    buf.flip(); // 将buffer切换到读的状态

    while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // 读取buffer 中的数据
    }

    buf.clear(); // 清空buffer
    bytesRead = inChannel.read(buf); 、
  }
  aFile.close();
```

这里注意 _buf.flip()_ 的调用，首先从 channel 中读取数据到 buffer ，再调用 flip() 去切换状态，然后才能读取 buffer 中缓存的数据，关于 buffer 的更多细节将在后续章节中进行介绍。

参考：
<br><http://tutorials.jenkov.com/java-nio/channels.html>
<br><http://ifeve.com/channels/>
