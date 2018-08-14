# Path

- [Path 简介](#path-简介)
- [Path 的创建](#path-的创建)
  - [Absolute Path 的创建](#absolute-path-的创建)
  - [Relative Path 的创建](#relative-path-的创建)
- [Path.normalize() 方法](#pathnormalize-方法)

### Path 简介

JAVA Path接口是NIO2中的一部分，是java6、java7中对NIO的更新。java Path接口在java7中加入nio，Path接口位于java.nio.file包下，所以其全限定名称为java.nio.file.Path。

一个java Path示例代表一个文件系统路径，这一路径即可指向文件，也可指向文件夹。路劲可以为绝对路径，也可为相对路径。一个绝对路径包含从系统根目录开始指向某个文件或文件夹的全路径。而相对路径为相对其他路径而指向某个文件或文件夹的路径。相对路径听起来好像有点混乱。不用担心，我会在接下来的教程中对相对路径做更多的解释。

不要混淆了文件系统路径和系统环境变量中的path路径，java.nio.file.Path 与系统环境变量中的path没有任何关系。

在很多方面，java.nio.file.Path接口与java.io.File 类很相似。但他们还是有几处主要的不同。在大多数情况下，你可以是用Path接口来代替File类的使用。

### Path 的创建

要是用java.nio.file.Path实例前，必须先创建一个Path实例。你可以通过Paths（java.nio.file.Paths）类中的Paths.get()这一静态方法来创建Path实例。以下为一个java Paths.get()方法调用的示例：

```
import java.nio.file.Path;
import java.nio.file.Paths;

public class PathExample {
  
  public static void mian(String[] args) {
    Path path = Paths.get("c:\\data\\myfile.txt");
  }
}
```

注意示例顶端的两个_import_语句，在使用Path接口和Paths类之前需要先引入他们。

其次，注意Path.get()方法的调用。调用Path.get()方法将创建一个Path示例。换句话说就是Path.get()方法为Path示例的工厂方法。

##### Absolute Path 的创建

通过调用以文件绝对路径为参数的Paths.get()这一工厂方法创建一个Absolute Path 实例对象。以下为创建一个绝对路径的实例对象的代码示例：

```
Path path = Paths.get("c:\\data\\myfile.txt");
```

绝对路径为：c:\data\myfile.txt，在java 的string中，两个\字符是必须的。因为\ 是一个转义字符，表示紧随他的字符需要被转义。通过\\的方式实际上就是告诉编译器这里就是一个\。

上面的地址是一个windows文件地址。在UNIX系统中，绝对路径可能看起来如下：

```
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

##### Relative Path 的创建
### Path.normalize() 方法


参考：<http://tutorials.jenkov.com/java-nio/path.html>
