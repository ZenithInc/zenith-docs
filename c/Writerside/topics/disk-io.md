# 文件的输入输出

在编程中，我们经常会讲数据写入到磁盘的动作称之为持久化或者落盘。也就是将数据从内存中写入到磁盘中，这样就不会因为断电而丢失数据。因此，对文件的读写至关重要。此处所说的文件就是字节的序列。

但是这里所说的文件仅仅是磁盘中保存的文件吗？对于类 Unix 系统来说，一切皆文件。对于这样的系统而言，文件的概念就更为广泛，也更为重要了。所以，这一篇文档将会详细描述 C 语言中的文件的输入和输出。

### 打开关闭文件 {id="open-closed"}

文件是什么呢？在 C 语言中，是一个名为 `File` 的结构体。至于这个结构体中包含有什么，那么不同的平台会有不同的实现。另外，在 C 语言中，为不同类型的文件的读写提供了相同的接口，基本的使用如下:
```c
char filepath[] = "/Users/sujinzhi/CLionProjects/CDemo/images/test.jpg";
FILE *file = fopen(filepath, "r");		// 打开文件，如果文件不存在则会返回 NULL
if (!file) {
	perror("");		// 输出错误信息
}
fclose(file); 	// 关闭文件
```
首先是我们需要“打开”一个文件，传入两个参数，一个是文件的路径，另一个是文件的打开模式。文件的打开模式就是一些有着具体含义的字符串，如下表所示:

| 访问模式字符 | 含义                    | 解释       | 如果文件存在 | 如果文件不存在 |
|--------|-----------------------|----------|--------|---------|
| r      | 读(Read)               | 打开文件并且可读 | 从头读取   | 打开失败    |
| w      | 写(Write)              | 创建文件并可写  | 清空原有内容 | 创建新文件   |
| a      | 追加(Append)            | 追加到文件末尾  | 在末尾写入  | 创建新文件   |
| r+     | 可扩展读(Read Extended)   | 打开文件读写   | 从头读取   | 错误      |
| w+     | 可扩展写(Write Extended)  | 创建文件读写   | 清空原有内容 | 创建新文件   |
| a+     | 可扩展追加(Write Extended) | 打开文件读写   | 在末尾写入  | 创建新文件   |

> 注意: + 号表示更新模式。


### 文件流的缓冲 {id="stream-buffer"}

我们的程序如果在读取文件的时候，需要由 CPU 给 DMAC(**D**ynamic **M**emory **A**ccess **C**ontroller) 一条命令，由它从磁盘中读取一个字符，然后存入内存，直到文件被全部读取。如下图所示:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/0tQi4iSIVYY1tK8ztbOR.png)

这样的处理方式，效率低下。所以，我们就在内存中开辟一块区域，称之为缓冲区。然后依然由 DMAC 不断地将数据从磁盘中写入到内存中的缓冲区内，当缓冲区满了之后，CPU 一次性从中读取数据即可。如下图所示:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/qDgQEnLdjcQBU7PEXZNk.png)

缓冲区的大小根据编译器的不同也有所区别，一般为 512 Byte 或者 4 KB。当然，也可以由开发者自定义缓冲区的大小，如下示例:
```c
char filepath[] = "/Users/sujinzhi/CLionProjects/CDemo/images/test.jpg";
FILE *file = fopen(filepath, "r");
// 设置缓冲区
char buf[8191];
setvbuf(file, buf, _IOLBF, 8192);
```
前两行代码上文中已经说过，是打开一个文件。末尾两行设置缓冲区，使用函数 `setvbuf` 。这个函数有三个参数，分别如下:

- 第一个参数：文件的具柄
- 第二个参数：文件的缓冲区
- 第三个参数：是否开启缓冲。这个参数有三个取值，分别如下：
    - _IOFBF：表示全量缓冲，不理会文件中的换行符
    - _IOLBF：表示按行缓冲
    - _IONBF：表示不使用用缓冲区

**在设置文件缓冲区的时候，要注意缓冲区变量的生命周期一定要比文件流的生命周期更长。**具体来说，示例代码中的 `buf` 变量一定要比 `file` 变量活得更久。你想啊，如果 `buf` 变量都销毁了，那么 `file` 文件流还往这个变量中写入数据，不就错了吗？下面就是一个错误的示例:
```c
char std_buffer[BUFSIZ];		// BUFSIZ 是一个宏，默认的缓冲区大小，可能是 512 可能是  4096
// stdout 这个流是从程序开始到程序结束，所以一定比 set_buffer 长命
// 需要注意的是，setbuf 这个函数已经不建议使用了，实际编程中，请使用上文中的 setvbuf, 可以指定缓冲区大小
setbuf(stdout, std_buffer);	
```
默认情况下，缓冲区满了之后才会清空缓冲区。但是，也可以手动地情况缓冲区，如下示例:
```c
fflush(file);
```

### 读写一个字符 {id="read-a-char"}

接着，我们来说说如何从文件中读取一个字符，分别介绍三个函数:

- **第一个函数是 `getchar` ：**从标准输入流 `stdin` 中读取一个字符

- **第二个函数是 `getc` : **从指定的输入流中读取一个字符， `getc(stdin)` 就相当于 `getchar()`

- **第三个函数是 `fgetc` : **在大多数情况下和 `getc` 函数是一样的，区别在于 `getc` 是宏实现，而 `fgetc` 是一个真正的函数。所以有些情况下 `getc` 函数会快一些，避免了堆栈调用，但是不能是带有副作用的表达式，比如说 `++i` 对参数递增，而 `fgetc` 则可以。

我们来使用 `getchar` 函数写一个回显字符的小程序，程序代码如下:
```c
while (1) {		// 通过死循环不断地读取输入的字符
	int c = getchar();		// 从 stdin 中读取一个字符
	if (c == EOF) {			// EOF 是一个整数，不同的编译环境也不一样，表述输入结束
		break;
	} else if (c == '\n') {	// 如果是换行就继续
		continue;
	}
	printf("%c", c);		// 回显字符
}
```
既然有从标准输入流(STDIN)读一个字符，也会有从标准输出流(STDOUT)输出一个字符。也会用到三个函数，和上面三个函数也是一一对应的，如下:

- **第一个函数是 `puchar` ：**向 STDOUT 输出指定的一个字符, 如果写入失败会返回 `EOF` 

- **第二个函数是 `putc` ：**向指定的输出流写入一个字符, `putc(c, stdout)` 等价于 `putchar(c)`

- **第三个函数是 `fputc` : **向指定的输出流写入一个字符

所以，上面示例中的 `printf("%c", c);` 也可以改写成  `putchar(c);` 。最后，我们来写一个程序：读取一个文件的内容并输出到控制台中:
```c
// 路径可以自行修改
char filepath[] = "/Users/sujinzhi/CLionProjects/CDemo/CMakeLists.txt";
FILE *file = fopen(filepath, "r");		// 打开一个文件，返回文件描述符
int c = getc(file);		// 获取文件的第一个字符
while(EOF != c) {		// 通过循环来遍历文件内容
	putchar(c);			// 输出读取到的字符
	c = getc(file);		// 再获取一个字符
}
fclose(file);			// 关闭文件描述符
```

### 读写一行字符 {id="read-row"}

上面说的是读取一个字符，那可不可以读取一行字符呢？可以使用函数 `gets` :
```c
char result[1024];
gets(result);
printf("%s\n", result);
```
但是当你的程序中包含这个函数并编译的时候，会给出如下警告, 说使用这个函数是不安全的。因为这个函数是根据换行符来决定一行的内容的，可能会读不完。

> warning: this program uses gets(), which is unsafe.


那有没有安全的替代方案呢？有的，可以使用 `fgets` 函数:
```c
char result[1024];
fgets(result, 1024, stdin);	// 如果输入的内容超出了 1024，则会截取
printf("%s", result);
```
既然有 `fgets` 用来获取输入，就会有对应的输出的函数，既 `fputs` 函数:
```c
char result[4];
fgets(result, 4, stdin);
fputs(result, stdout);
```

### 按照字节读写 {id="read-bytes"}

之前的案例中，我们写了一个回显字符的例子。第一次我们是按字符来读写的，第二次重构我们按照行来读写，这一次我们使用字节来读写，每次读写固定的字节数。

介绍两个函数，分别是 `fread` 和 `fwrite` ，我们使用这两个函数来改写之前的例子:
```c
int buffer[BUF_SIZE];		// 定义一个整型数组，用来存储读取的字节
while (1) {
    // 分别传入 buffer、每个 buffer 元素的大小、读取的总数以及文件流, 得到实际读取的字节大小
	size_t bytes_read = fread(buffer, sizeof(buffer[0]), BUF_SIZE, stdin);
    // 如果实际读取的字节大小小于最大的 BUF_SIZE, 分情况判断，是一行读取完毕还是发生了错误
	if (bytes_read < BUF_SIZE) {
		if (feof(stdin)) {
            // 一行读取完毕，将读取的内容输出
			fwrite(buffer, sizeof(buffer[0]), bytes_read, stdout);
			break;
		} else if (ferror(stdin)) { /** 发生错误，错误处理 **/ } 
		break;
    }
    // 读取一整行，等于或者大于 BUF_SIZE，直接输出
	fwrite(buffer, sizeof(buffer[0]), BUF_SIZE, stdout);
}
```
那么，我们定义的 `buffer` 变量里存储的是什么内容呢？我们在终端输入 `1234567890abcdef` , 然后通过 DEBUG，我们得到了这个数组里的内容:

| buffer[0] | buffer[1] | buffer[2]  | buffer[3]  |
|-----------|-----------|------------|------------|
| 875770417 | 943142453 | 1650536505 | 1717920867 |

这些数字是十进制的，我们将其转换成十六进制，方便观察。比如说 `875770417` 转成十六进制是 `0x34333231` ，这里是采用小端序存储的，所以从后往前读分别是 31、32、33、34，这个在 ASCII 码中分别对应的是 1、2、3、4。这下子明白了吧， `fread` 和 `fwrite` 两个函数是成对出现的，文件实际上都是按照二进制存储的，读写也是按照固定字节读、按照固定字节写的。

### 案例:复制文件 {id="copy-file"}

有了上面的内容铺垫，那么写出一个复制文件的程序就非常容易了，直接把示例程序贴出来:
```c
int CopyFile(const char *src_filepath, const char *dist_filepath) {
    FILE *src_file = fopen(src_filepath, "r");
    if (!src_file) {/* 省略错误处理 */}
    FILE *dist_file = fopen(dist_filepath, "w");
    if (!dist_file) fclose(src_file);	// 关闭文件
    while (1) {			// 循环遍历文件
        int c = getc(src_file);
        if (EOF == c) {		// 文件读取到末尾返回 EOF，则退出循环
            break;
        }
        putc(c, dist_file);	// 将读取到的字符写入目标文件
    }
    // 关闭两个文件的描述符
    fclose(src_file);
    fclose(dist_file);
    return 0;
}
```
上面是按照字符来读取，那么可不可以按行来读取呢？因为按行读取的效率肯定是比按字符性能更好。删除上面示例中的第 6 行到第 12 行，换成如下版本:
```c
while (1) {
	char buffer[1024];
	char *next = fgets(buffer, 1024, src_file);
	if (!next) break;
	if (!fputs(buffer, dist_file)) { /** 写入失败，错误处理 **/ }
}
```
但是上面的示例中，我们通过 `!next` 判断是否结束，如果结束就退出，这样的做法并不严谨。因为 `next` 除了文件读写完成之外，还可能是文件读写错误，所以我们要继续判断属于哪一种情况:
```c
if (!next) {
 	if (ferror(src_file)) {
        /** 文件错误，错误处理 **/ 
    } else if (feof(src_file)) {
        /** 文件读取完毕，后续处理 **/ 
    } else {
        /** 一定要加上这个 else, 反之出现其他错误，也可以在此处添加日志，方便排查 **/
    }
}
```
具体来说，为什么按行读取会比按字符读取性能更好呢？这个因为编译平台的不同答案也是不一样的，比如说在 MSVC 中，按字符是读一个字符加一次锁，按行是读一行字符加一次锁。而 GCC 在默认情况下是不加锁的，所以性能的差异并不是非常明显。

最后，我们再来重构一次，按照字节来读写文件, 将示例中的 `while` 部分改写如下:
```c
while (1) {
	size_t bytes_read = fread(buffer, sizeof(buffer[0]), 4, src_file);
	if (bytes_read < 4) {
		if (feof(src_file)) {
			fwrite(buffer, sizeof(buffer[0]), bytes_read, dist_file);
			break;
		} else if (ferror(src_file)) { /** 错误处理 **/ } else { /** 其他错误 **/ }
        break;
    }
    fwrite(buffer, sizeof(buffer[0]), bytes_read, dist_file);	// 省略错误处理
}
```

### 重定向 {id="redirect"}

在 Linux 中，一个非常有用的能力叫做重定向，一个程序的输出可以作为一个程序的输入。那么在 C 语言中，怎么实现重定向呢？可以使用函数 `frepoen` :
```c
freopen("/Users/sujinzhi/CLionProjects/CDemo/test.log", "a", stdout);
puts("Hello World!");
fclose(stdout);
```
执行这个函数之后，标准输出会被重定向到指定的文件中。也就是说，这个程序执行完成之后，在控制台中不会输出任何内容，“Hello World”将会被输入到指定的文件中。

那么如果我希望再次重定向呢？使用标准库就没有办法了，不过我们可以使用 `posix` 标准，当中有两个函数，分别是 `dup` 和 `dup2` 函数。它们可以完成这件事情, 我们来封装一下这个重定向的函数。

首先需要引入这些函数的头文件，不同的平台也不一样，我们可以使用条件编译:
```c
#if defined(__APPLE__) || defined(__linux__)
#    include <unistd.h>
#elif defined(_WIN32)
#    include <io.h>
#endif
```
来封装这个函数:
```c
void RedirectStdout(char const *filename) {
    static int saved_stdout_no = -1;        // 用来记录上一次保存的文件描述符
    if (filename) {
        // 表示之前都没有调用重定向,
        if (saved_stdout_no == -1) {
            // fileno 函数可以获取文件描述符，然后使用
            saved_stdout_no = dup(fileno(stdout));
        }
        fflush(stdout);         // 刷新缓冲区
        freopen(filename, "a", stdout);
    } else {
        // 说明已经重定向过了, 恢复到基本输出
        if (saved_stdout_no != -1) {
            fflush(stdout);     // 刷新缓冲区
            dup2(saved_stdout_no, fileno(stdout));
            close(saved_stdout_no);
            saved_stdout_no = -1;
        }
    }
}
```
测试这个函数:
```c
puts("Hello ");
RedirectStdout("/Users/sujinzhi/CLionProjects/CDemo/test.log");
puts("World ");
RedirectStdout(NULL);
puts("!");
```

### 案例：统计文件字符个数 {id="wc"}

这个案例展示了如何统计一个文件中的字符个数。首先，我们需要定义一些错误的宏:
```c
#define ERROR_ILLEGAL_FILENAME -1
#define ERROR_CANNOT_OPEN_FILE -2
#define ERROR_READ_FILE -3
#define ERROR_UNSUPPORTED_CHARSET -4
```
统计文件的字符个数要区分字符编码，这个案例支持 GBK 和 UTF8 两种编码，定义这两种字符编码的宏:
```c
#define CHARSET_UTF8 0
#define CHARSET_GBK 1
```
接着我们来封装统计函数, 代码如下:
```c
unsigned long CountChartersInFile(char const *filename, int charset) {
    // 判断文件名是否为空，如果为空则直接返回错误
    if (!filename) return ERROR_ILLEGAL_FILENAME;
    // 定义文件的指针类型
    FILE *file;
    // 根据编码类型的不同，分别设置编码、打开文件
    switch (charset) {
        case CHARSET_GBK:
            // 根据不同的平台条件编译，实现跨平台
#ifdef _WIN32
            setlocale(LC_ALL, "chs");
#else
            setlocale(LC_ALL, "zh_CN.gbk");
#endif
            file = fopen(filename, "r");
            break;
        case CHARSET_UTF8:
            setlocale(LC_ALL, "zh-CN.utf-8");
#ifdef _WIN32
            file = fopen(filename, "r, ccs=utf-8");
#else
            file = fopen(filename, "r");
#endif
            break;
        default:
            return ERROR_UNSUPPORTED_CHARSET;
    }
    if (!file) return ERROR_CANNOT_OPEN_FILE;
#define BUFFER_SIZE 512
    wchar_t wcs[BUFFER_SIZE];
    unsigned long count = 0;
    // 遍历文件统计字数
    while (fgetws(wcs, BUFFER_SIZE, file)) {
        unsigned long length = wcslen(wcs);
        count += length;
    }
    // 判断错误
    if (ferror(file)){
        fclose(file);
        return ERROR_READ_FILE;
    }
    fclose(file);
    return count;
}
```
到这里就写好了。
