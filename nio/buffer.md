# 缓冲区 Buffer 

java NIO 中的缓冲区(buffer)是用来与NIO中的通道(channel)进行交互的。如你所知，数据可从通道(channel)读取到缓冲区(buffer)，也可将缓冲区(buffer)中的数据写入到通道(channel)。

缓冲区(buffer) 实际上就是一块你可以先写入数据，然后再从中读取数据的内存区域。这块内存区域被封装成了NIO Buffer对象，其提供了很多方法来方便ni 操作这块内存区域。

### 缓冲区(buffer)的基本使用：

使用一个缓冲区(buffer)去读写数据，通常分为以下4个小步骤：

1. 将数据写入buffer
2. 调用flip()方法
3. 从buffer中读取数据
4. 调用clear()或者compact()方法

当你向缓冲区(buffer)中写入数据时，缓冲区(buffer)会记录下你写入了多少数据。一旦你需要从中读取数据，你需要将调用缓冲区的flip()方法将其从写模式切换到读模式。在读模式下你可读取你写入的全部数据。

一旦你读完缓冲区(buffer)中的全部数据，你需要清空缓冲区(buffer)以保证其能再次被写入。你有2中方式去实现这一过程：调用clear()方法或调用compact()方法。clear()方法将清空整个缓存区(buffer)中的全部数据，而compact()方法仅仅清除你已经读取过得数据，未读的数据将会移动到缓冲区(buffer)的首部，再次写入数据时将在未读数据后面继续追加。

下面是一个简单的 buffer 使用示例：

```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); // 读取数据缓存到buffer.
while (bytesRead != -1) {

  buf.flip();  // 切换buffer到读模式下

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // 每次从buffer中读取1个字节 
  }

  buf.clear(); //清空buffer确保下次能再次写入
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

### 缓冲区(buffer)的容量(Capacity)，位置(Position)和限制(Limit)

缓冲区(buffer) 实际上就是一块你可以先写入数据，然后再从中读取数据的内存区域。这块内存区域被封装成了NIO Buffer对象，其提供了很多方法来方便你操作这块内存区域。

想要搞清楚缓冲区(buffer)是如何工作的必须要弄明白他的三个重要属性，分别是：

- 容量 Capacity
- 位置 Position
- 限制 Limit

位置(position)和限制(limit)的含义取决缓冲区(buffer)是在读还是写模式下。
容量(capacity)的含义不会因缓冲区(buffer)的模式而改变，都是表示缓冲区(buffer)的总容量

下图是位置(position)、容量(capacity)和限制(limit)在读和写模式下的含义示意图，会在本文中后文对这三个属性再做详细解释:

![](./pic/buffers-modes.png)

##### 容量(Capacity)

作为一个内存区域，缓冲区(buffer)有一个固定的大小，称作它的容量(capacity)。你至多能向缓冲区(buffer)中写入容量(capacity)个byte|short|char|int|long|float|double。一旦缓冲区(buffer)满了，继续写入之前必须先将其清空。

##### 位置(Position)

当你将数据写入缓冲区(buffer)的时候，会写在一个确切的位置(position)。起初这个位置为0，当你写入一个byte|short|char|int|long|float|double到缓冲区(buffer)后，位置(position)将会自动移动到缓冲区(buffer)的下一个可写入数据的内存单元，位置(position)最大值为容量(capacity) -1

当你从缓冲区(buffer)中读取数据时，位置(position)同样是一个给定的值。当你调用flip()将缓冲区(buffer)从写模式切换到读模式下的时候，位置(position)将会重置为0。当你从当前位置(position)读取过数据后，位置(position)会自动移动到下一个可读的位置。

##### 限制(Limit)

当缓冲区(buffer)处在写模式下时，限制(Limit)就是指你最多可以向缓冲区(buffer)中写入多少数据。即写模式下，limit=capacity
当缓冲区(buffer)切换到读模式下时，限制(Limit)便是指你最多可从缓冲区(buffer)中读取多少数据。因此，当缓冲区(buffer)从写切换到读后，limit将等于写模式下的position的值。换句话说就是你写入多少数据就能读多少(limit将会设置成已被写入的数据数量，即写模式下的position值)

### 缓冲区(Buffer)类型

java NIO 中提供了以下Buffer类型：

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

如你所见，缓冲区(buffer)类型代表不同的数据类型，换句话说，就是可以通过byte|char|short|int|long|float|double类型来操作缓冲区中的字节。
<br>_MappedByteBuffer_ 有点特殊，在涉及到它的专门的章节中再做介绍。



参考：
<br><http://tutorials.jenkov.com/java-nio/buffers.html>
<br><http://ifeve.com/buffers/>
