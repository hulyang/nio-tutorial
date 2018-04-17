# 分散和聚合 Scatter AND Gather

目录
- [Scatter&Gather 简介](#scattergather-简介)
- [Scattering Reads](#scattering-reads)
- [Gathering Writes](#gathering-writes)

### Scatter&Gather 简介

java NIO 你开始就内置了scatter|gather的支持，scatter|gather是对channel进行读写的一种概念。

分散读(scattering read)，就是指将一个channel中的数据读取到一个或多个buffer的一组读取操作。因此，channel将自身的数据"分散"(scatter)到了多个buffer中。

聚合写(gathering write)，就是指将一个或多个buffer中的数据全写入到单个channel中的一组操作。因此，channel"聚合"(gather)了多个buffer中的数据。



### Scattering Reads

### Gathering Writes
