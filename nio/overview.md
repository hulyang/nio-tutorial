# 概述

  java nio 包目录大概如下:
	
	nio
	├── channels	selector以及各种channel
	├── charset		
	├── file
	└── 各种buffer
  
其中核心部分为:

  - channel		位于channels包下
  - buffer		位于nio包下
  - seletor		位于channels包下


### Channel 和 Buffer

基本上，所有的 IO 在 NIO 中都是从一个 channel 开始。 channel 有点像流（但是流是单向的， channel 却是双向的），数据可以从 channel 读到 buffer ，也可从 buffer 写到 channel 。下面有张图示:

![](../pic/overview-channels-buffers1.png)
