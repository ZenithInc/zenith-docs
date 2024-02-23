# JAVA Stream IO

当涉及到处理数据输入和输出时，Java IO（Input/Output）是每个Java程序员必须熟悉的关键概念之一。无论是读取文件、处理网络数据，还是与外部设备进
行通信，Java IO提供了丰富的工具和类库，使这些任务变得更加容易。

```plantuml
@startuml
class Object

class File
class RandomAccessFile
class InputStream
class OutputStream
abstract class Reader
abstract class Writer

File -up-> Object
RandomAccessFile -up-> Object
InputStream -up-> Object
OutputStream -up-> Object
Reader -up-> Object
Writer -up-> Object

class FilterInputStream
class ObjectInputStream

FilterInputStream -up-> InputStream
ObjectInputStream -up-> InputStream

class FilterOutputStream

FilterOutputStream -up-> OutputStream

class InputStreamReader

InputStreamReader -up-> Reader

class OutputStreamWriter

OutputStreamWriter -up-> Writer

class BufferedInputStream

BufferedInputStream -up-> FilterInputStream

class BufferedOutputStream

BufferedOutputStream -up-> FilterOutputStream

class FileReader

FileReader -up-> InputStreamReader

class FileWriter

FileWriter --> OutputStreamWriter

@enduml
```

## 什么是 Java IO {id="what"}

如上文中的 UML 图示，Java IO（Input/Output) 是处理磁盘、网络、内存数据输入输出的机制和类库。其作用如下:

```plantuml
@startmindmap
+ Java IO
++ 应用场景
+++ 文件操作
+++ 网络通信
+++ 标准输入输出
+++ 内存操作
+++ 对象序列化
-- 风格
--- 传统 IO
--- NIO
-- 类型
--- 字节流（Byte Stream)
--- 字符流（Character Stream）
@endmindmap
```

## 基本输入输出流 {id="basic-io-streams"}

无论是从文件中读取数据还是将数据写入到不同的目标，我们都应该先了解基本的 IO 流。这一小节中，我们将讲解如下内容:
```plantuml
@startmindmap
* 基本输入输出流
    ** 输入流和输出流的概念
    ** 字节流和字符流的区别
    ** InputStream 和 OutputStream 的基本用法
    ** Reader 和 Writer 的基本用法
    ** 读写文件的示例
@endmindmap
```

### 输入流和输出流的概念 {id="input-stream-and-output-stream"}

我们先来说，什么是流。在 Java 中，我们经常将输入输出流和 Lamdba 中的流混为一谈，其实它们是两个完全不同的术语。Lamdba 中的 Stream 指的是处理
集合和数据流的操作，比如 `map`、`filter`、`reduce`，是函数式编程的一部分，通常用来对 `List` 、`Set`、`Map` 等集合进行操作。

而我们接下去要说的 Streams，指的是对文件、网络、内存中的数据，从一端流向另一端。

* **输入流**：它指的是将外部的数据读取到 Java 的程序中。比如从文件、网络、内存等外部输入数据。
* **输出流**：它指的是从 Java 程序中输出数据到文件、网络、内存、屏幕等。

### 字节流和字符流的区别 {id="difference-byte-streams-and-character-streams"}

我们上文已经提到了，IO 流有两种类型，分别是字节流和字符流。它们的区别如下表所示:

| 方面   | 字节流             | 字符流           |
|------|-----------------|---------------|
| 数据类型 | 处理原始字节数据        | 处理字符数据        |
| 数据单位 | 以字节为单位          | 以字符为单位        |
| 字符编码 | 不关心字符编码         | 考虑字符编码和字符集    |
| 适用场景 | 用于处理二进制数据和非文本文件 | 用于处理文本文件和字符数据 |

它们两者之间，常见的类如下图所示:

```plantuml
@startmindmap
+ 常见的类
++ 字节流
+++ FileInputStream
+++ FileOutputStream
+++ DataInputStream
-- 字符流
--- FileReader
--- FileWriter
--- BufferedReader
--- BufferedWriter
@endmindmap
```

### InputStream 和 OutputStream 基本用法 {id="input-stream-and-output-stream-usage"}

我们先来看 `InputStream` 的 UML 图，从整体上有一个清晰的认识:

```plantuml
@startuml
class InputStream
class PipedInputStream
class FileInputStream

PipedInputStream -down-> InputStream
FileInputStream -down-> InputStream

class FilterInputStream

FilterInputStream -up-> InputStream

class PushbackInputStream

PushbackInputStream -up-> FilterInputStream

class DataInputStream

DataInputStream -down-> FilterInputStream

interface DataInput

DataInputStream --|> DataInput

class StreamTokenizer
class Reader

StreamTokenizer *- Reader : have 1 >
StreamTokenizer *- InputStream : have 1 >

@enduml
```

然后，我们再来看 `OutputStream` 的 UML 图:

```plantuml
@startuml
class OutputStream

class PipedOutputStream 
class FileOutputStream

PipedOutputStream -down-> OutputStream
FileOutputStream -down-> OutputStream


@enduml
```