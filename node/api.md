# node API

## 1. path.join([path1][, path2][, ...]) 和 path.resolve([from ...], to)

path.join()方法可以连接任意多个路径字符串。要连接的多个路径可做为参数传入。path.join()方法在接边路径的同时也会对路径进行规范化。

path.resolve()方法可以将多个路径解析为一个规范化的绝对路径。其处理方式类似于对这些路径逐一进行cd操作，
与cd操作不同的是，这引起路径可以是文件，并且可不必实际存在（resolve()方法不会利用底层的文件系统判断路径是否存在，而只是进行路径字符串操作）。

即：对于以/开始的路径片段，path.join只是简单的将该路径片段进行拼接，而path.resolve将以/开始的路径片段作为根目录，在此之前的路径将会被丢弃，就像是在terminal中使用cd命令一样。

```
 path.join('/foo', 'bar') // '/foo/bar' 
 path.join('/foo', 'bar', 'baz/asdf', 'quux', '..') // '/foo/bar/baz/asdf' 

path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile') // D:\tmp\subfile
path.resolve('/foo/bar', './baz') // '/foo/bar/baz' 
path.resolve('/foo/bar', '/tmp/file/') // '/tmp/file' 
path.resolve('/foo/bar', './tmp/file/') // '/foo/bar/tmp/file' 
```