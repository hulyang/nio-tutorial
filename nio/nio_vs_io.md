# NIO VS IO

- [NIO VS IO 简介](#nio-vs-io-简介)
- [NIO与IO的主要区别](#nio与io的主要区别)
- [面向stream与面向buffer](#面向stream与面向buffer)
- [阻塞与非阻塞IO](#阻塞与非阻塞io)
- [选择器Selector](#选择器selector)
- [NIO与IO对程序设计的影响](#nio与io对程序设计的影响)
  - [API的调用](#api的调用)
  - [数据处理](#数据处理)
- [总结](#总结)

### NIO VS IO 简介

当学习了JAVA NIO和IO 之后，有个问题值得我们思考：

什么时候使用传统IO？什么时候使用NIO？

在本文中，我会尽力揭露NIO于IO的差异、NIO于IO的使用以及他们如何影响程序设计方面。

### NIO与IO的主要区别

下表总结了NIO与IO之间的主要区别。我会在后面章节中详细的阐述表中说提到的差异：

| IO | NIO |
| - | - |
| 面向流(stream) | 面向缓冲(buffer) |
| 阻塞IO | 非阻塞IO |
|  | 选择器Selector |

### 面向stream与面向buffer

JAVA NIO 与IO 最大的不同就是IO是面向流的，而NIO是面向缓冲的。这又意味着什么呢？

JAVA IO面向流，就意味着

### 阻塞与非阻塞IO
### 选择器Selector
### NIO与IO对程序设计的影响
##### API的调用
##### 数据处理
### 总结
