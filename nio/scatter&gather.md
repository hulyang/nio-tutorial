# Scatter AND Gather

目录
- [Scatter&Gather 简介](#scattergather-简介)
- [Scattering Reads](#scattering-reads)
- [Gathering Writes](#gathering-writes)

### Scatter&Gather 简介

java NIO 你开始就内置了scatter|gather的支持，scatter|gather是对channel进行读写的一种概念。

分散读(scattering read)，就是指将一个channel中的数据读取到一个或多个buffer的一组读取操作。因此，channel将自身的数据"分散"(scatter)到了多个buffer中。

聚合写(gathering write)，就是指将一个或多个buffer中的数据全写入到单个channel中的一组操作。因此，channel"聚合"(gather)了多个buffer中的数据。

scatter|gather在处理分段传输的数据时，是有很大优势的。例如，例如一条信息有头部(header)和主体(body)组成，你就可以将header和body分别保存在不同的buffer中，然后你就能更好的去分开处理header和body。

### Scattering Reads

scattering read 是指将数据从单个channel读取到多个buffer中的过程。下面是scatter原理示意图：

![](./pic/scatter.png)

下面是执行scattering read的示意代码：

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = {header, body};
channel.read(bufferArray);
```

注意，你需要先将buffer放入数组当中，然后将数组作为参数，传递给channel.read()方法，read()方法将按照数组中元素的顺序依次向buffer()中写入channel中的数据，一旦buffer满了，就会自动切换到下一个继续写入。

事实上，scattering read 再向下一个buffer中写入数据之前必须要将当前的buffer填满，意味着在消息各部分大小不确定的时候是很不适用的。换句话说，如果消息中有header和body，并且header大小是固定的(如128byte)，那scattering read 将会很适用。

### Gathering Writes

gathering write 是将多个buffer中的数据写入到单个channel中的过程。下面是gather原理示意图：

![](./pic/gather.png)

下面是gathering write的示意代码：

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = {header, body};
channel.write(bufferArray);
```

buffer数组作为参数传递给writer()方法，将会按照数组中元素顺序依次向channel中写入buffer中的内容。只有position和limit之间的数据内容(remain部分)会被写入。因此，若一个buffer的capacity为128字节，但是仅包含58个字节，那么就只会向channel中写入58个字节。因此，和scattering read 不同，gathering write 在处理消息各部分大小不确定时仍然适用。

参考：
<br><http://tutorials.jenkov.com/java-nio/scatter-gather.html>
<br><http://ifeve.com/java-nio-scattergather/>
