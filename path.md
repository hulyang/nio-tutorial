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

相对路径是指基于某个路劲指向另外一个文件或目录的路径。将确定路径和相对路径拼接起来就是相对路径对应的绝对路径地址。

JAVA NIO Path Class统一可以使用相对路径。你可以通过调用Paths.get(basePath, relativePath)的方式创建一个相对路径。以下是2个再java中创建相对路径的例子：

```
Path projects = Paths.get("d:\\data", "projects");

Path file     = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

第一个例子创建了一个指向 d:\data\projects文件夹的java Path实例。第二个例子创建了一个指向d:\data\projects\myfile.txt文件的实例。

在使用相对路劲时，有如下两种特殊的字符供你使用：

- .
- ..

.字符表示的就是当前路径。例如，如果你像下面这样来创建一个相对路径：

```
Path currentDir = Paths.get(".");
System.out.println(currentDir.toAbsolutePath());
```

所创建的java Path实例相应的绝对路径就是这段代码的项目工程目录。

如果.被用在路劲字符串中间，就表示依然是当前路劲下。以下是相应的代码示例：

```
Path currentDir = Path.get("d:\\data\\projects\\.\\a-project");
```

这一路径对应的就是：d:\\data\\projects\a-project。

..表示的是父目录或者说上一级目录。以下是响应的代码示例：

```
Path parentDir = Paths.get("..");
```

示例中创建的这一path实例对应的路径就是这段代码所在工程目录的父目录。

而当你在路径字符串中间看到..的时候那就是指向当前目录的上一级目录。例如：

```
String path = "d:\\data\\projects\\a-project\\..\\another-project";
Path parentDir2 = Paths.get(path);
```

示例中创建的这一path实例对应的绝对路径为：d:\data\projects\another-project

a-project后面的..将路劲变化为其父目录projects然后再指向其中的another-project目录。
 
.和..也可以组合的使用在双参数的Paths.get()方法中。以下是两个简单的java Paths.get示例：
 
```
Path path1 = Paths.get("d:\\data\\projects", ".\\a-project");

Path path2 = Paths.get("d:\\data\\projects\\a-project", "..\\another-project");
```

有很多的JAVA NIO Path类去使用相对路劲的方式，你可以在后续的教程中继续学习。
 
### Path.normalize() 方法

Path中的normalize()方法可以标准化路径。标准化的意思就是移除路径中所包含的.以及..并最终指向正确的路劲地址。以下是一个nomalize()方法调用的示例：

```
String originalPath = "d:\\data\\projects\\a-project\\..\\another-project";

Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);

Path path2 = path1.normalize();
System.out.println("path2 = " + path2);
```

以上示例中首先创建了一个包含..的路劲字符串，然后通过这一字符串创建了一个path1实例并将其打印输出。

示例然后调用path1的normalize()方法来返回一个新的Path实例path2，再将返回的这一新的路劲打印输出。

下面是以上示例的输出：

```
path1 = d:\\data\\projects\\a-project\\..\\another-project
path2 = d:\\data\\projects\\another-project
```

如你所见，标准化的路劲是不包含a-project\..这部分的，因为他是多余的，移除的部分加入进去对最终的绝对路劲不会有任何影响。

参考：<http://tutorials.jenkov.com/java-nio/path.html>
