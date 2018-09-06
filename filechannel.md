# FileChannel

- [FileChannel 简介](#filechannel-简介)
- [FileChannel的打开](#filechannel的打开)
- [FileChannel的数据读取](#filechannel的数据读取)
- [FileChannel的数据写入](#filechannel的数据写入)
- [FileChannel的关闭](#filechannel的关闭)
- [FileChannel Position](#filechannel-position)
- [FileChannel Size](#filechannel-size)
- [FileChannel Truncate](#filechannel-truncate)
- [FileChannel Force](#filechannel-force)

### FileChannel 简介

java NIO FileChannel是用于连接文件的一个channel，使用FileChannel，你可以从文件中读取数据，也可写入数据到文件中，java NIO FileChannel类，是读写文件的java 标准IO的一种代替方式。

FileChannel不可设置成非阻塞的模式，它永远是阻塞的IO。

### FileChannel的打开

在你使用FileChannel之前，你必须先打开它。你不可以直接打开一个FileChannel。你必须通过一个InputStream，OutputStream或RandomAccessFile去获得它。以下是通过RandomAccessFile获得FileChannel的一个示例：

```
RandomAccessFile aFile = new RandomAccessFile("aFile.txt", "rw");

FileChannel inChannel = afile.getChannel();
```

### FileChannel的数据读取

你可调用某个read()方法来从FileChannel中读取数据。以下为代码示例：

```
ByteBuffer buf = ByteBuffer.allocate(48);

int byteRead = inChannel.read(buf);
```

首先的分配一个buffer，数据是从FileChannel中读取到buffer中的。

然后，调用FileChannel.read()方法将数据从FileChannel读取到buffer中。该方法的返回值(int类型)，告诉我们有多少数据写入了buffer中，如果返回-1，说明已经到了文件末尾。

### FileChannel的数据写入

向FileChannel中写入数据是使用write()方法，该方法将buffer作为参数。以下为代码示例：

```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();
while (buf.hasRemain()) {
    channel.write(buf);
}
```

注意，FileChannel.write()方法是在一个while循环中去掉用的。这是因为每次调用write()方法时，我们是不能确定每次想FileChannel中写入多少数据的，因此我们需要不断重复的调用write()方法直到buffer中已经没有数据可以继续写入。

### FileChannel的关闭

当你用完一个FileChannel后记得将其关闭，下面是示例代码：

```
channel.close();
```

### FileChannel Position

当你对一个FileChannel进行读写操作时，都是基于某个position，你可以通过调用FileChannel对象的position()方法来获得其当前的position。

你也可以通过调用FileChannel的position(long pos)方法来设置它的当前position。

以下为代码示例：

```
long pos = channel.position();

channel.position(pos + 123);
```

### FileChannel Size

FileChannel的size()方法返回的是其所连接的文件大小。以下为代码示例：

```
long fileSize = channel.size();
```

### FileChannel Truncate

你可调用FileChannel的truncate()方法来截取(truncate)文件，当你truncate文件时，你得指定一个截取的长度。以下为代码示例：

```
channel.truncate(1024);
```

以上代码截取了文件1024个字节。

### FileChannel Force

FileChannel.force()方法会将channel中所有的未写入的数据写入到磁盘上。出于性能的考虑，操作系统可能将数据缓存到内存中，所以你不能保证写入到channel中的数据确实写入到了磁盘上，除非你调用了force()方法。

force()方法有个boolean类型的参数，指明是否需要将文件的元数据(权限等...)也写入到磁盘上。

以下为同时写入数据和元数据的一个示例：

```
channel.force(true);
```

参考：<http://tutorials.jenkov.com/java-nio/file-channel.html>
