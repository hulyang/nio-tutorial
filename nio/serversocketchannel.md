# ServerSocketChannel

目录
- [ServerSocketChannel 简介](#serversocketchannel-简介)
- [ServerSocketChannel的打开](#serversocketchannel的打开)
- [ServerSocketChannel的关闭](#serversocketchannel的关闭)
- [ServerSocketChannel监听连接](#serversocketchannel监听连接)
- [非阻塞模式](#非阻塞模式)


### ServerSocketChannel 简介

java NIO ServerSocketChannel是用来监听TCP新连接的一类channel，就像java标准的Network中的ServerSocket那样。ServerSocketChannel类位于java.nio.channels包下。

以下为示例代码：

```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

### ServerSocketChannel的打开

你可以通过ServerSocketChannel.open()方法来打开一个ServerSocketChannel，以下为代码示例：

```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

### ServerSocketChannel的关闭

你可以通过ServerSocketChannel.close()方法来关闭一个ServerSocketChannel，以下为代码示例：

```
serverSocketChannel.close();
```

### ServerSocketChannel监听连接

通过调用ServerSocketChannel.accept()方法，可以监听新进来的连接。然accept()方法返回时，他会返回一个关于这次连接的SocketChannel对象。因此，accept()方法会阻塞到有连接到达为止。

因为你通常不会只关心单次连接，所以accept()方法通常会像下面这样,放在一个while循环中调用：

```
while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

当然你会有其他的让循环终止的判断条件来代替示例代码中while循环中的true;

### 非阻塞模式

ServerSocketChannel可以被设置成非阻塞模式。在非阻塞模式下，accept()方法会立即返回，因此如果没有新的连接建立的话，返回值为null。所以你就需要去检查返回值是否为null。以下为代码示例：

```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
        }
}
```

参考：<http://tutorials.jenkov.com/java-nio/server-socket-channel.html>
