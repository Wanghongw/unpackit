# UnpackIt

### 原项目

本项目在下面这个项目做了自定制的改动:[c4milo/unpackit](https://github.com/c4milo/unpackit)

### 原项目存在的问题

当我们使用gzip命令直接打包一个文件为.gz文件的话，如果原文件的大小大于4K，那么解压出来的文件只有4K大小。

本人测试了下：使用gzip命令压缩一个100K大小的json文件，解压后的文件只有4K，`4K后面的数据会丢失`。

### 原因

原因就是，原项目在解压直接打包为gz文件的压缩文件时使用了bufio.NewReader这个方法：

```go
 decompressingReader = bufio.NewReader(gzr)
```

看一下官方源码就知道了，默认情况下只给缓冲区设置4K的大小:

```go
const (
    defaultBufSize = 4096
)

// NewReader returns a new Reader whose buffer has the default size.
func NewReader(rd io.Reader) *Reader {
    return NewReaderSize(rd, defaultBufSize)
}
```

### 改造后的包的用法

改一下里面的源码就可以了，改造以后的包的用法如下所示。

其中最后一个参数设置的是缓冲区的大小，具体根据你解压之后的文件大小而定。

#### Unpack a file

```go
    file, _ := os.Open(test.filepath)
    destPath, err := unpackit.Unpack(file, tempDir, "unpackFileName", 4*1024)
```

#### Unpack a stream (such as a http.Response):

```go
    res, err := http.Get(url)
    destPath, err := unpackit.Unpack(res.Body, tempDir, "unpackFileName", 4*1024)
```

