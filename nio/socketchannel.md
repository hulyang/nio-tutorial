# SocketChannel

- [SocketChannel的简介](#socketchannel的简介)
- [SocketChannel的打开](#socketchannel的打开)
- [SocketChannel的关闭](#socketchannel的关闭)
- [SocketChannel的读取](#socketchannel的读取)
- [SocketChannel的写入](#socketchannel的写入)
- [非阻塞模式](#非阻塞模式)
  - [connect()](#connect)
  - [write()](#write)
  - [read()](#read)
  - [Non-blocking Mode with Selectors](#non-blocking-mode-with-selectors)


### SocketChannel的简介

java NIO SocketChannel是连接TCP网络套接字的一个channel类型，是等价于[java Networking's Sockets](http://tutorials.jenkov.com/java-networking/sockets.html)的
java NIO。有两种创建SocketChannel的方式：

1. 你可以打开一个SocketChannel并连接至因特网上的某处的一个服务器。
2. 当一个新连接到达ServerSocketChannel时将会创建一个SocketChannel。

### SocketChannel的打开

以下为打开SocketChannel的代码示例：

```
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

### SocketChannel的关闭

你可以调用SocketChannel.close()方法来关闭SocketChannel，以下为代码示例：

```
socketChannel.close();
```

### SocketChannel的读取

你可调用某个read()方法来从SocketChannel中读取数据。以下为代码示例：

```
ByteBuffer buf = ByteBuffer.allocate(48);

int byteRead = socketChannel.read(buf);
```

首先分配一个buffer，我们从SocketChannel读取数据到这个buffer中。

然后，调用SocketChannel.read()方法，这个方法就会从SocketChannel中读取数据到buffer中。方法的返回值(int)表明有多少数据写入了buffer中。如果返回-1，说明已经到达了流的末尾，或者连接已经关闭。

### SocketChannel的写入

你可以使用SocketChannel.write()方法向SocketChannel中写入数据，该方法有一个参数buffer。以下为代码示例：

```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

注意，SocketChannel.write()方法是在一个while循环中去调用的。这是因为每次调用write()方法时，我们是不能确定每次向SocketChannel中写入多少数据的，因此我们需要不断重复的调用write()方法直到buffer中已经没有数据可以继续写入。

### 非阻塞模式

你可以将SocketChannel设置为非阻塞模式(Non-blocking Mode)。当你这么做以后，就可以异步的调用connect(),write(),read()方法了。

##### connect()

如果SocketChannel处于非阻塞模式下，然后你调用connect()方法，该方法可能会在建立连接之前就给你返回了。为了确定连接是否建立，你可以像下面这样调用finishConnect()方法：

```
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(!socketChannel.finishConnect()){
    //wait, or do something else...    
}
```

##### write()

在非阻塞状态下，write()方法可能可能啥都没写就已经返回。因此，你需要在循环中去调用write()方法，但是前面已经有过代码示例了，这里就不再重复了。

##### read()

在非阻塞状态下，read()方法可能啥都没读取到就返回了。因此你的主要read()方法的返回值(int)，它会告诉你读取到了多少数据。

##### Non-blocking Mode with Selectors

SocketChannel的非阻塞模式搭配Selector一起使用效果会更好，将多个SocketChannel注册到同一个Selector，可以通过询问Selector来了解哪些Channel已经有事件(read、write等)准备就绪。更多关于Selector和SocketChannel的使用细节会在教程的后续章节中再做说明。

参考：<http://tutorials.jenkov.com/java-nio/socketchannel.html>
