# DatagramChannel

目录
- [DatagramChannel 简介](#datagramchannel-简介)
- [DatagramChannel的打开](#datagramchannel的打开)
- [接收数据](#接收数据)
- [发送数据](#发送数据)
- [连接指定地址](#连接指定地址)

### DatagramChannel 简介

JAVA NIO DatagramChannel是一种可接收和发送UDP数据包的channel。因为UDP是种无连接的网络协议，所以你不能像其他channel一样直接从中读写数据，而是收发数据包。

### DatagramChannel的打开

以下是打开DatagramChannel的代码示例：

```
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```

这段示例打开了一个可接收端口在9999上的UDP数据包的datagramchannel。

### 接收数据

你可以像以下示例这样，通过调用datagramchannel的receive()方法来接收数据：

```
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();

channel.receive(buf);
```

receive()方法会将接收到的数据包内容复制到给定的buffer中。如果接收到的数据包大小超过了buffer的容量，超出部分将会被舍弃。

### 发送数据

你可以像以下示例这样，通过调用datagramchannel的send()方法来发送数据：

```
String newData = "New string to write to file ..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80);
```

这段示例代码将string发送给了"jenkov.com" 服务器 UDP 80端口。但是服务器并未监听该端口，所以什么都不会发生。你不会被告知发出去的数据包是否被接收，因为UDP 本身对数据的传输就没有任何保证。

### 连接指定地址

将Datagramchannel连接到网络上特定的地址是可行的。但是因为UDP 是无连接的，这种与特定地址的连接方式，并不是想TCP 那样创建一个真实的连接，而是将datagramchannel锁定在特定的地址上，使它只会从特定地址收发数据。

以下为代码示例：

```
channel.connect(new InetSocketAddress("jenkov.com",80));
```

当连接好了以后，你就可以像使用其他传统channel一样来调用read()和write()方法。只是在数据传输上不会有任何保证。以下为简单示例：

```
int bytesRead = channel.read(buf);
```

```
int bytesWrite = channel.write(buf);
```
