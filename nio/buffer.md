# Buffer

目录
- [Buffer 简介](#buffer-简介)
- [Buffer 的基本使用](#buffer-的基本使用)
- [Buffer 的 capacity、position、limit、mark](#buffer-的-capacitypositionlimitmark)
  - [capacity](#capacity)
  - [position](#position)
  - [limit](#limit)
  - [mark](#mark)
- [Buffer 类型](#buffer-类型)
- [Buffer 的分配](#buffer-的分配)
- [Buffer 写操作](#buffer-写操作)
- [flip() 方法](#flip-方法)
- [Buffer 读操作](#buffer-读操作)
- [rewind() 方法](#rewind-方法)
- [clear() 和 compact() 方法](#clear-和-compact-方法)
- [mark() 和 reset() 方法](#mark-和-reset-方法)
- [equals() 和 compareTo() 方法](#equals-和-compareto-方法)
  - [equals() 方法](#equals-方法)
  - [compareTo() 方法](#compareto-方法)

### Buffer 简介

java NIO 中的buffer是用来与NIO中的channel进行交互的。如你所知，数据可从channel读取到buffer，也可将buffer中的数据写入到channel。

buffer实际上就是一块你可以先写入数据，然后再从中读取数据的内存区域。这块内存区域被封装成了NIO Buffer对象，其提供了很多方法来方便你操作这块内存区域。


  
### Buffer 的基本使用

使用一个buffer去读写数据，通常分为以下4个小步骤：

1. 将数据写入buffer
2. 调用flip()方法
3. 从buffer中读取数据
4. 调用clear()或者compact()方法

当你向buffer中写入数据时，buffer会记录下你写入了多少数据。一旦你需要从中读取数据，你需要将调用缓冲区的flip()方法将其从写模式切换到读模式。在读模式下你可读取你写入的全部数据。

一旦你读完buffer中的全部数据，你需要清空buffer以保证其能再次被写入。你有2中方式去实现这一过程：调用clear()方法或调用compact()方法。clear()方法将清空整个buffer中的全部数据，而compact()方法仅仅清除你已经读取过得数据，未读的数据将会移动到buffer的首部，再次写入数据时将在未读数据后面继续追加。

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

### Buffer 的 capacity、position、limit、mark

buffer实际上就是一块你可以先写入数据，然后再从中读取数据的内存区域。这块内存区域被封装成了NIO Buffer对象，其提供了很多方法来方便你操作这块内存区域。

想要搞清楚buffer是如何工作的必须要弄明白他的四(教程中为前三个，mark是我添加的)个重要属性，分别是：

- 容量 Capacity
- 位置 Position
- 限制 Limit
- 位置 Mark

> Buffer源码中关于这4个成属性，有行很重要的注释，我觉得有必要列出来的，这也方便大家对buffer中这四个属性能有个更好的理解,注释内容为：

> Invariants: mark <= position <= limit <= capacity

> 这行注释诠释了这四个属性永远满足的一个条件。buffer中很多方法(基本都为为属性赋值的方法)都会检查这一条件。

position和limit的含义取决buffer是在读还是写模式下。
capacity的含义不会因buffer的模式而改变，都是表示buffer的总容量

下图是position、capacity和limit在读和写模式下的含义示意图，会在本文中后文对这三个属性再做详细解释:

![](./pic/buffers-modes.png)

##### Capacity

作为一个内存区域，buffer有一个固定的大小，称作它的capacity。你至多能向buffer中写入capacity个byte|short|char|int|long|float|double。一旦buffer满了，继续写入之前必须先将其清空。

##### Position

当你将数据写入buffer的时候，会写在一个确切的position。起初这个位置为0，当你写入一个byte|short|char|int|long|float|double到buffer后，position将会自动移动到buffer的下一个可写入数据的内存单元，position最大值为capacity-1

当你从buffer中读取数据时，position同样是一个给定的值。当你调用flip()将buffer从写模式切换到读模式下的时候，position将会重置为0。当你从当前position读取过数据后，position会自动移动到下一个可读的位置。

##### Limit

当buffer处在写模式下时，Limit就是指你最多可以向buffer中写入多少数据。即写模式下，limit=capacity
当buffer切换到读模式下时，Limit便是指你最多可从buffer中读取多少数据。因此，当buffer从写切换到读后，limit将等于写模式下的position的值。换句话说就是你写入多少数据就能读多少(limit将会设置成已被写入的数据数量，即写模式下的position值)

##### Mark
mark对于buffer的整体结构以及读写操作并不会产生什么影响，可能这也是原教程中没有拿出来介绍的原因。
mark顾名思义就是一个标记位，记录下某一个特殊位置，然后可以再重新倒回到这一特殊位置。关于这部分功能本文后续章节会再做介绍。


### Buffer 类型

java NIO 中提供了以下Buffer类型：

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

如你所见，buffer类型代表不同的数据类型，换句话说，就是可以通过byte|char|short|int|long|float|double类型来操作缓冲区中的字节。
<br>_MappedByteBuffer_ 有点特殊，在涉及到它的专门的章节中再做介绍。

### Buffer 的分配

要想获得一个buffer必须先分配它，每种类型的buffer都有一个allocate()方法来完成这一操作，下面是分配一个可存储48字节(capacity=48)的ByteBuffer的示例：

```
ByteBuffer buf = ByteBuffer.allocate(48);
```

再来一个可存储1024个字符的(capacity=1024)CharBuffer的分配示例：

```
CharBuffer buf = CharBuffer.allocate(1024);
```

基本上分配方式都大同小异，在明确存储类型了调用相关类型Buffer的allocate()方法来获得相应类型的buffer即可。

### Buffer 写操作

你有两种方式将数据写入buffer

1. 将数据从channel写入到buffer
2. 直接通过buffer自身的put()方法写入数据

下面是从channel写入数据到buffer的示例：

```
int byteRead = inChannel.read(buf); // 读取channel数据，写入到buffer
```
下面是通过put()方法写入数据到buffer中的示例：
```
buf.put(127);
```

### flip() 方法

flip()将buffer从写模式切换到读模式，调用flip()方法将会把position设置为0，将limit设置为之前position的值。

换句话说就是position现在为读的位置，而limit为写入buffer的数据总量，即可以被读取的数据总数。

> 原教程讲的比较拗口，感觉这里不如直接贴个源码简单明了(jdk1.8)：

```
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

> 源码中明显看出clip()除了改变了limit和position，另外将mark重新设置为了-1(即无标记位置的状态)

### Buffer 读操作

同样也有两种方式从buffer中读取数据：

1. 将buffer中的数据读取到channel
2. 直接调用buffer自身的的某个get()方法读取数据

下面是将buffer中数据读取到channel的示例：

```
int byteWrite = inChannel.write(buf); // 读取buffer数据，写入到channel
```

下面是通过get()方法来从buffer中读取数据

```
byte aByte = buf.get();
```

get()方法有很多其他版本，允许你以不同方式从buffer中读取数据。例如：读取某个特定位置的数据，亦或是从buffer中读取一个字节数组。更多的buffer实现细节参考javaDoc。

### rewind() 方法

buffer.rewind()方法将position设置成0，所以你可以重新冲buffer中读取数据，rewind()方法是不改变limit的，因此limit仍然表示你可以从buffer中读取多少数据。

> 以上是对于方法的解释，基本上讲的挺明白的了，不过还是直接上源码更加清晰(jdk1.8)：

```
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

> 源码中明显看出rewind()除了将position倒退回去，另外将mark重新设置为了-1(即无标记位置的状态)

### clear() 和 compact() 方法

一旦你从buffer中读取了数据后，你可以调用clear()方法或者compact()方法，来确保buffer被能再次写入。

如果你调用clear()方法，position将会设置为0，limit将会设置为capacity的值。换句话说就是buffer被clear后，其实buffer中所保存的数据是没有被清除的，仅仅是改变了能再次写入数据的标志位。

当我们调用clear()方法时，如果buffer中有数据没有读取，那些数据将会被遗忘(并没被清除)，意味着你不再能知道哪里的数据被读了，哪里的数据还没有度。

所以如果buffer中还有未读的数据，并且等会需要读取，但是现在得先往buffer中写入部分数据，此时应该调用compact()而非clear()

compact()方法将所有的未读数据拷贝到buffer首部，然后将position设置为最后一个未读元素的下一个位置，limit属性还是会像clear()方法一样设置为capacity的值。现在buffer便可写入新数据，而且不会覆盖未读的数据。

> 教程中说了这么多，我们来直接贴代码，等会做个简单的小结。

> clear()方法：

```
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```

> compact()方法 (Buffer类中并未定义compact()方法，此为HeapByteBuffer类中该方法的实现的，注释为本人添加)：

```
public ByteBuffer compact() {
    // ByteBuffer中是使用一个名为hb的byte数组做数据存储的，remaining()方法返回的是 limit-position，即未读的数据长度
    System.arraycopy(hb, ix(position()), hb, ix(0), remaining()); // 先将未读部分拷贝到存储区域首部
    position(remaining()); // 修改position值为未读数据的下一位
    limit(capacity()); // 将limit值改为capacity的值
    discardMark(); // 清除mark，即mark=-1
    return this;
}
```

> 简单来说clear()和compact()都可以将buffer从读模式下切换到写模式，compact()会将未读的数据移到buffer首都等待下次读取，再次写入数据会接着这部分后面继续写，而clear()不会保留未读数据，再次写入数据时会将未读部分覆盖。而不管compact()和clear()方法都只是仅仅移动position和limit而已，并没有去清空buffer中所保留的数据。

### mark() 和 reset() 方法

你可以通过调用buffer.mark()方法标记当前的位置，然后过后可以通过buffer.reset()方法来让position返回原先标记的位置。以下为示例代码：

```
buffer.mark();  // public final ByteBuffer mark() { mark = position; return this;}

//call buffer.get() a couple of times, e.g. during parsing.

buffer.reset();  //set position back to mark.   
```

> mark()很简单，就是将position值赋值给mark教程中的解释也很简洁明了，我在示例代码中通过注释加入了方法实现，就不再贴出源码中的实现了。
reset()方法也很简单，就是将mark再赋值给position，只不过多了一遍mark合法性的检查。代码如下(jdk1.8)：
```
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```

### equals() 和 compareTo() 方法

有时候你可能需要调用equals()或者compareTo()来比较两个buffer。

##### equals() 方法

> equals()方法继承自Object类，compareTo()方法是因为实现了Comparable接口。其实现过程都是在各自的子类中做具体实现的，但是实现逻辑都是一致。所以下面我们不会贴出源码，仅仅阐述比较的逻辑。

两个buffer相等的条件为：

1. 他们类型相同
2. 他们所剩余的未读数据量相同
3. 他们所剩的未读数据一一相等

如你所见，equals()方法仅仅对比了buffer中的部分内容，并没比较buffer中的每一个元素。事实上只是比较了buffer中未读的那部分。

##### compareTo() 方法

compareTo()方法也是比较两个buffer中的未读部分，应用在一些需要排序的场景下。
<br>当满足下列条件时我们一个buffer比另外一个buffer要小：

1. 当两个buffer中相对应位置的未读数据不等时，数据小的那个buffer要小
2. 当所有相对位置的未读数据都相等时，先耗尽的那个buffer要小(即未读部分的数据量小的个buffer要小)



参考：
<br><http://tutorials.jenkov.com/java-nio/buffers.html>
<br><http://ifeve.com/buffers/>
