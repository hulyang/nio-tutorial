# Pipe

- [Pipe 简介](#pipe-简介)
- [Pipe的创建](#pipe的创建)
- [Pipe的写入](#pipe的写入)
- [Pipe的读取](#pipe的读取)

### Pipe 简介

JAVA NIO Pipe 是两个线程之间的单向数据连接。一个Pipe 包含一个source channel和一个sink channel，你向sink channel中写入数据，这部分数据将可以在source channel 中进行读取。

以下是一个Pipe 的简单图示：

![](./pic/pipe-internals.png)

### Pipe的创建

你可以通过调用Pipe.open()方法来打开一个pipe，以下为代码示例：

```
Pipe pipe = Pipe.open();
```

### Pipe的写入

向Pipe 中写入数据，你得先获取到他的sink channel，以下为代码示例：

```
Pipe.SinkChannel sinkChannel = pipe.sink();
```

然后你就可以像下面这样通过调用sinkChannel的write()往里面写入数据：

```
String newData = "New String to write to file ..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.put(newData.getBytes());
buf.clear();

buf.flip();

while(buf.hasRemaining() {
  sinkChannel.write(buf);
}
```

### Pipe的读取

从pipe 中读取数据，你得先获得他的source channel。以下为代码示例：

```
Pipe.SourceChannel sourceChannel = pipe.source();
```

然后你就可以像下面这样通过调用sourcechannel 的read()方法来从中读取数据：

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = sourceChannel.read(buf);
```

int 类型的返回值表示从中读取了多少字节的数据。

参考：<http://tutorials.jenkov.com/java-nio/pipe.html>
