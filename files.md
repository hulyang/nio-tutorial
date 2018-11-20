# Files

- [Files 简介](#files-简介)
- [Files.exists()](#filesexists)
- [Files.createDirectory()](#filescreatedirectory)
- [Files.copy()](#filescopy)
  - [覆盖已存在的文件](#覆盖已存在的文件)
- [Files.move()](#filesmove)
- [Files.delete()](#filesdelete)
- [Files.walkFileTree()](#fileswalkfiletree)
  - [搜索文件](#搜索文件)
  - [递归删除文件目录](#递归删除文件目录)
- [Files类中的其他方法](#files类中的其他方法)

### Files 简介

JAVA NIO Files 类，提供了一些方法去处理系统文件。这篇教程将覆盖这些方法的大多数通常情况下的使用。Files 类包含很多方法，所以，当你需要某个这里没有阐述的方法，去检查JavaDoc，Files类可能有你所要的方法的。

java.nio.file.Files 类需要配合 java.nio.file.Path 示例来使用，所以在使用Files类之前，需要先理解Path类。

总的来说这就是工具类。为我们提供一些简单的文件操作，例如：

- 判断文件是否存在
- 创建目录
- 拷贝文件
- 移动文件
- 删除文件
- 遍历文件、目录
- 等等

### Files.exists()

Files.exists()方法用于检查指定Path路径是否存在于文件系统当中。

创建一个文件系统中不存在的Path示例是完全可以的。例如，如果你想要创建一个新的目录，那你必须先创建其相对应的Path实例，然后创建目录。

因此，Path实例是有可能指向于不存在文件系统的文件路径，你可以使用Files.exists()方法来查看该路径是否正是存在。

以下是Files.exists()使用实例：
```
Path path = Paths.get("data/logging.properties");

boolean pathExists = Files.exists(path, new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
```

上面示例代码先创建了一个Path示例指向我们想检查是否真是存在的文件路径，然后将Path实例作为Files.exists()的第一个参数来调用该方法。

得注意Files.exists()方法的第二个参数，这个参数是一个操作数组，这组操作将会影响Files.exists()去判断该路径是否存在。上面示例代码中的数组包含了__LinkOption.NOFOLLOW_LINKS__，意味着Files.exists()方法将不会遵循文件系统中的符号连接去判断路径是否存在。

### Files.createDirectory()

Files.createDirectory() 方法将会根据指定的Path示例来创建一个新的目录，以下是Files.createDirectory()方法的简单示例：
```
Path path = Paths.get("data/subdir");

try {
    Path newDir = Files.createDirectory(path);
} catch(FileAlreadyExistsException e){
    // 目录已经存在.
} catch (IOException e) {
    // 其他IO异常
    e.printStackTrace();
}
```

第一行代码创建了一个Path实例指代马上要创建的目录，在 try-catch 代码块中，使用这一Path实例作为参数来调用Files.createDirectory()方法，如果成功创建目录，将会返回一个新的Path示例指向新创建的文件路径。

如果该目录已经存在，则会抛出java.nio.file.FileAlreadyExistsException异常，如果发生了一些其他的错误，将会抛出一个IOException异常。例如，要创建的目录的父目录不存在的情况，则会抛出一个IOException异常。

### Files.copy()

Files.copy方法将一个文件拷贝到另外一个文件，以下是一个简单的Files.copy()示例：

```
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath);
} catch(FileAlreadyExistsException e) {
    //destination file already exists
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```

示例中首先创建了拷贝源文件和目标文件的Path实例，然后将这两个Path实例作为参数传入Files.copy()方法，这样就会将源文件的文件内容拷贝至目标文件当中。

如果目录文件已经存在，则会抛出java.nio.file.FileAlreadyExistsException异常，如果发生了一些其他的错误，将会抛出一个IOException异常。例如目标文件的父目录并不存在，则会抛出IOException异常。

##### 覆盖已存在的文件

有时候也会需要强制拷贝并覆盖已存在的文件，以下是使用Files.copy()去强制拷贝并覆盖已有文件的示例：
```
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath, StandardCopyOption.REPLACE_EXISTING);
} catch(FileAlreadyExistsException e) {
    //destination file already exists
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```
注意Files.copy()方法的第三个参数。这一参数命令copy()方法，如果目标文件已存在，则覆盖已存在的目标文件。

### Files.move()

JAVA NIO Files类包含一个方法，可以将一个文件移动到另外一个路径，移动文件其实就等同于对其重命名，移动文件修改文件目录的同时对该文件重命名。是的，java.io.file 类调用renameTo()同样可以做到。

以下是Files.move()方法示例：
```
Path sourcePath      = Paths.get("data/logging-copy.properties");
Path destinationPath = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.move(sourcePath, destinationPath, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
    //moving file failed.
    e.printStackTrace();
}
```

示例中首先创建了源文件路径和目标文件路径，然后调用Files.move()方法，该文件将会移动为目标文件路径。（可以理解为windows操作下的剪切，重命名）

注意Files.move()方法的第三个参数，该参数会命令Files.move()方法去覆盖已存在的目录文件。当然，这个参数是可选的。

Files.move()方法可能会抛出IOException异常，例如目标文件已经存在，但你并没有设置__StandardCopyOption.REPLACE_EXISTING__，又或者要移动的源文件不存在等等。

### Files.delete()

Files.delete()方法可以删除一个文件或者目录，以下是Files.delete()方法的示例：

```
Path path = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.delete(path);
} catch (IOException e) {
    //deleting file failed
    e.printStackTrace();
}
```
First the Path pointing to the file to delete is created. Second the Files.delete() method is called. If the Files.delete() fails to delete the file for some reason (e.g. the file or directory does not exist), an IOException is thrown.
示例中首先创建一个Path指向要删除的文件路径，然后调用Files.delete()方法。如果Files.delete()方法由于某些原因调用失败（例如文件或目录不存在），则会抛出一个IOEXception异常。

### Files.walkFileTree()

Files.walkFileTree()方法将递归遍历一个目录。walkFileTree()需要一个Path实例和一个FileVisitor作为参数。Path实例指向你所需要递归遍历的目录，FileVisitor将在每次访问时调用。

在我解释递归遍历目录如果工作之前，先贴一下FileVisitor接口：

```
public interface FileVisitor {

    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException;

    public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {

}
```

你的去实现自定义的FileVisitor接口，然后将你的实现作为参数传递给 walkFileTree()方法。你所实现的FileVisitor的每个方法将会被调用在递归目录的不同时刻。如果你不需要实现其全部方法，你可以继承 SimpleFileVisitor类，该类已经默认实现了FileVisitor接口中的所有方法。


以下是一个简单的 walkFileTree()示例：

```
Files.walkFileTree(path, new FileVisitor<Path>() {

	@Override
	public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
		System.out.println("pre visit dir:" + dir);
		return FileVisitResult.CONTINUE;
	}

	@Override
	public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
		System.out.println("visit file: " + file);
		return FileVisitResult.CONTINUE;
	}

	@Override
	public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
		System.out.println("visit file failed: " + file);
		return FileVisitResult.CONTINUE;
	}

	@Override
	public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
		System.out.println("post visit directory: " + dir);
		return FileVisitResult.CONTINUE;
	}
});
```

FileVisitor 中实现的每个方法，会在递归遍历的不同时刻去掉用：

preVisitDirectory()方法仅在访问目录前调用。postVisitDirectory() 方法仅在访问目录结束后调用。

visitFile()会在每次访问文件时调用，访问目录是不会调用，仅仅访问文件才会调用。visitFileFailed()方法会在访问文件失败时调用。例如，你没有文件访问权限，或者一些其他的错误。

这四个方法都会返回一个FileVisitResult枚举，FileVisitResult包含以下四个选项：

- CONTINUE
- TERMINATE
- SKIP_SIBLINGS
- SKIP_SUBTREE

通过放回这四个枚举值其中之一，去告诉方法遍历是否需要继续。

CONTINUE 意味着遍历正常继续。

TERMINATE 意味着遍历将会终止。

SKIP_SIBLINGS 意味着遍历将会继续，但是不会继续访问任何此文件的同级其他文件。

SKIP_SUBTREE 意味着遍历将会继续，但是不会进入这个目录。这个值仅有在preVisitDirectory()中返回才有意义，如果其他方法中返回这个值将被视为CONTINUE。

##### 搜索文件

可以利用 walkFileTree()方法来搜索文件，例如继承SimpleFileVisitor 去搜索名为 README.txt 的文件：

```
Path rootPath = Paths.get("data");
String fileToFind = File.separator + "README.txt";

try {
	Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
    
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      String fileString = file.toAbsolutePath().toString();
      //System.out.println("pathString = " + fileString);

      if(fileString.endsWith(fileToFind)){
        System.out.println("file found at path: " + file.toAbsolutePath());
        return FileVisitResult.TERMINATE;
      }
      return FileVisitResult.CONTINUE;
    }
  });
} catch(IOException e){
    e.printStackTrace();
}
```

##### 递归删除文件目录

Files.walkFileTree()也可以用来递归删除目录中的文件和该目录。Files.delete()方法仅在目录为空是才能删除该目录，通过删除每个目录下的所有文件文件（在visitFile()方法中），然后删除目录本身（在postVisitDirectory()），你就可以删除一个有子目录和子文件的目录了。下面是递归删除的代码示例：
```
Path rootPath = Paths.get("data/to-delete");

try {
	Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
	@Override
	public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
		System.out.println("delete file: " + file.toString());
		Files.delete(file);
		return FileVisitResult.CONTINUE;
	}

	@Override
	public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
		Files.delete(dir);
		System.out.println("delete dir: " + dir.toString());
		return FileVisitResult.CONTINUE;
	}
	});
} catch(IOException e){
	e.printStackTrace();
}
```

### Files类中的其他方法

java.nio.file.Files类中还有很多其他有用的方法，例如创建符号链接，计算文件大小，设置文件权限等，可以自行去阅读JavaDoc去了解这些方法的详细信息。

参考：<http://tutorials.jenkov.com/java-nio/files.html>