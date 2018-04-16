# 通道 channel

通道(channel) 类似流，但是又有些许不同:
- 一个 channel 既可读，亦可写，但是流都是单向的(只读|只写)
- 通道可以异步读写
- 通道总是读写到 buffer 
如上所述，你可将channel 中的数据读取到 buffer, 也可将 buffer 中的数据 写入到 channel , 下面是张很好的示例图:

![](./pic/overview-channels-buffers1.png)

