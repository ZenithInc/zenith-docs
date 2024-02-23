# 字符串处理

这一篇开始，我们将详细讲解在 C 语言中的字符串的处理应用，比如字符串和其他数值的转换、长度和比较、拆分、复制和连接等。

### 判断字符的类型 {id="check-string-type"}

在 C 语言的标准库中已经提供了很多的字符类型判断函数，它们都包含在头文件中 `ctype.h` 中:

| 单字节      | 宽字节       | 描述                                   |
|----------|-----------|--------------------------------------|
| isalnum  | iswalnum  | 是否为字母数字                              |
| isalpha  | iswalpha  | 是否为字母                                |
| islower  | iswlower  | 是否为小写字母                              |
| isupper  | iswupper  | 是否为大写字母                              |
| isdigit  | iswdigit  | 是否为数字                                |
| isxdigit | iswxdigit | 是否为十六进制数字                            |
| isgraph  | iswgraph  | 是否为图形字符                              |
| isspace  | iswspace  | 是否为空格字符(包括制表符、回车符、换行符等)              |
| isblank  | iswblank  | 是否为空白字符(包括水平制表符)                     |
| isprint  | iswprint  | 是否为可打印字符                             |
| ispunct  | iswpunct  | 是否为标点符号                              |
| tolower  | towlower  | 转换成小写字母                              |
| toupper  | towupper  | 转换成大写字母                              |
| 不适用      | iswctype  | 检查一个 `wchar_t` 是否属于指定的分类             |
| 不适用      | towctrans | 使用指定的变换映射来转换一个 `wchar_t` (实际上是大小写转换) |
| 不适用      | wctype    | 返回一个宽字符的类别，用于 `iswctype` 函数          |
| 不适用      | wctrans   | 返回一个变换映射，用于 `towctrans` 函数           |

上面这些函数都是通过静态查表的算法实现的，所以相比对比 ASCII 值对比来说性能要高出很多。早期的 Linux 采用的是比较测试(Comparison tests)来实现的，比如:
```c
#define isdigit(x) ((x) >= '0' && (x) <= '9')
```
这样的方式存在潜在的错误，后期改用了静态查表的方式来实现，比如:
```c
#define isdigit(x) (TABLE[x]) & 1)
```

### 字符串转换成数值 {id="str-to-number"}

在实际的编程中，我们经常会需要将字符串转换成数值。在当前的 C 语言标准中，提供了两种类型的函数都可以实现它。分别是 `ato` 类和 `strto` 类。在实际的编程中，如果应用场景比较简单可以使用前者，如果应用场景更为复杂可以使用 `strto` 类，当然它的使用也会更加的复杂。

#### ato 系列函数 {id="ato"}

第一种是 `ato` 系列的函数，适用于一些比较简单的应用场景中:
```c
double	 atof(const char *);
int	 atoi(const char *);
long	 atol(const char *);
#if !__DARWIN_NO_LONG_LONG
long long
	 atoll(const char *);
#endif /* !__DARWIN_NO_LONG_LONG */
```
根据函数都可以见名知意，比如 `atoi` 就是将字符转换成整型数值(Alpha to Integer)。另外, `atoll` 是在 C99 标准之后引入的，因为这个标准加入了 `long long` 数值类型，所以使用了宏来判断是否启用这个函数。
```c
int i = atoi("1234");			// value is 1234
int ii = atoi("   1234abcd");	// value is 1234
int iii = atoi("0x10");			// 有的编译器可能不支持十六进制数值，iii is 0
```
对于 `atoi` 系列的函数而言，如果转换成功，则会输出指定的数值。反之如果转换失败，比如说超出数据类型的范围，会输出什么结果在 C 语言标准中是为定义的，不同的编译器有着不同的实现。

#### strto 系列函数(推荐) {id="strto"}

大部分情况下，我们都推荐使用 `strto` 系列的函数。下表列举了部分常用的函数:

| 函数名                   | 描述                                                                                       |
|-----------------------|------------------------------------------------------------------------------------------|
| strtol、strtoll        | 将字符串转换为有符号整型                                                                             |
| strtoul、strtoull      | 将字符串转换为无符号整型                                                                             |
| strtof、strtod、strtold | 将字符串转换为浮点型                                                                               |
| strtoimax、strtoumax   | 将字符串转换为所在环境中表示范围最大的整型 `intmax_t` 、无符号整型 `uintmax_t` , 与前面这些函数不同的是，这两个函数定义在 `stdint.h`  中 |

相比 `ato` 系列的函数，它使用复杂，但是可重复解析、更安全以及功能更为强大，比如说可以指定转换后的进制。我们来写一个示例，理解什么是可重复解析:
```c
char const *const str = "1024 2048 20000000000000000000000000000000000000000000000 3 -4 5 abcd bye";		// 定义一个包含了数值和非数值的字符串
char const *start = str;		// 开始指针
char *end;		// 结束指针
while(1) {		// 死循环重复解析，利用改变结束指针来实现的
   errno = 0;	// 如果解析失败，则会更改 errno,需要引入对应的头文件 error.h
   const long i = strtol(start, &end, 10);	// 传入开始指针和尾指针，10 是表示转换成十进制
   if (start == end) {		// 如果开始指针等于结束指针则表明转换结束，看第 15 行改变开始指针
       break;
   }
   // 下面 .* 的写法的含义是：可以在后面传入字符串的长度，而不以\0作为结束的标志
   printf("'%.*s'\t ==> %ld\n", (int)(end - start), start, i);
   if (errno == ERANGE) {
       perror("");		// 2000000...... 会导致转换数值范围溢出，所以会打印 Result too large
   }
   start = end;		// strto 系列函数内部会改变 end 指针的值，所以，通过这样的方式遍历整个字符串
}
```
这个程序的关键还在于第 7 行，我们传入了 `start` 指针和 `end` 指针，内部通过修改 `end` 指针来实现对字符串的遍历，这也就是所谓的重复解析。

### 字符串的长度与比较 {id="string-length"}

在 C 语言中，我们可以使用 `<string.h>` 头文件中的 `strlen` 来返回字符串的长度，如下示例:
```c
char str[] = "hello world!";
printf("%lu\n", strlen(str));	// Output: 12
```
我们知道在 C 语言中，使用 `\0` 来表示字符串的结尾，比如说下面这个例子:
```c
char str[] = "hello\0 world!";
printf("%lu\n", strlen(str));	// Output: 5
```
如果使用 `strlen` 来返回一些特别长的字符串就可以会一直统计下去。所以，在 C11 标准中又提供了  `strnlen` 函数来返回字符串的长度:
```c
char str[] = "hello\0 world!";
printf("%lu\n", strnlen(str, 4));	// Output: 4, 第二个参数表示最大计算长度
```
和 `strlen` 类似的是，C 语言也提供了字符串比较函数： `strcmp` 和 `strncmp` 两个版本，相较于前者多出了一个最大计算长度的参数:
```c
char *left = "Abc";
char *right = "Bcd";
printf("%d\n", strncmp(left, right, 3));	// Output: -1, 因为 B 大于 A
```

### 查找字符与子串 {id="research"}

我们先来看看如何在字符串中查找字符，C 语言提供了 `strchr` 和 `strrchr` 两个函数。前者指的是从前往后找，后者则是从后往前查找。请看示例:
```c
char str[] = "Hello World";
printf("%s\n", strchr(str, 'o'));   // Output: o World
printf("%s\n", strrchr(str, 'o'));  // Output: orld
printf("%d\n", (int)(strchr(str, 'o') - str)); // Output: 4
printf("%s\n", strchr(str, 'a')); // Output: null
```
另外，还提供了一个 `strpbrk` 的函数，可以查找多个字符，如下的示例遍历了一个字符串:
```c
char str[] = "Java, PHP, C; JavaScript;Lua;C++, Visual Basic";
char *p = str;
do {
	p = strpbrk(p, ",;");	// 通过改变 p 的指针来遍历字符串，返回查找到的位置
	if (p) {
		printf("%s\n", p);
        // 比如返回的字符串是 , PHP, C; JavaScript;Lua;C++, Visual Basic
        // 通过 p++ 就成了  PHP, C; JavaScript;Lua;C++, Visual Basic
		p++;	// 以此来避免死循环
	}
} while (p);
```
上面介绍的都是如何查找字符，那么如何查找字符串呢？使用函数 `strsub` 即可，就不贴代码了。

### 字符串的拆分 {id="explode"}

字符串根据指定的分隔符进行拆分，在实际编程中也非常的实用。如果是其他的高级语言，一般都会提供对应的函数或者方法，几行代码就能够搞定了。但是对于 C 语言来说，并不是一件简单的事情，还涉及到了动态内存的分配。C 语言的标准库提供了 `strtok` 函数来实现字符串的拆分。

首先，我们来定义一个将要被拆分的字符串，此处分隔符实用 `,` :
```c
char str[] = "Java,PHP,C,JavaScript,Lua,C++,Visual Basic";
```
接着，定义一个结构体，用来存放每一个拆分后的单词:
```c
typedef struct {
	char *name;
} Language;
```
因为我们并不知道会这个字符串会被拆分出多少个单词，所以需要动态分配 Language 的数组的内存:
```c
int language_capacity = 3;		// 数组的容量默认为 3，如果不足后面再行扩充
int language_size = 0;			// 拆分的语言的数量默认值为 0，每次拆分成功则递增
// 使用 malloc 函数分配初始大小，既结构体的大小乘数组默认的容量
Language *languages = malloc(sizeof(Language) * language_capacity);
```
接着，我们需要通过循环来拆分这个字符串:
```c
char *next = strtok(str, ",");		// 先拆分一次，如果没有遇到分隔符，则返回 NULL
while(next) {
	Language language;
	language.name = next;		// 每次拆分得到的字符串，也就是语言的名字
	// 判断 languages 内存是否足够，不够则开辟新内存
	if (language_size + 1 >= language_capacity) {
		language_capacity *= 2;			// 因为分配内存是有较大性能损耗的，所以每次扩容一倍
        // 使用 realloc 函数重新分配内存
		languages = realloc(languages, sizeof(Language) * language_capacity);
	}
	// 加入解析出来的单词
	languages[language_size++] = language;
    // strtok 内部保存了字符串的状态，所以传入 NULL 就可以了，所以这个函数也是线程不安全的
	next = strtok(NULL, ",");
}
```
最后我们来测试这段代码:
```c
printf("size is %d\n", language_size);
for (int i = 0; i < language_size; i++) {
	printf("name is %s\n", languages[i].name);
}
```

### 字符串的连接和复制 {id="cat-and-copy"}

在 C 语言中，要完成字符串的连接可以使用 `strcat` 或者使用 `strcpy` 函数。下面先展示前者:
```c
char src[] = " World";
char dest[20] = "Hello";
strcat(dest, src);		// 如果dest的空间不够，则 dest 为空
printf("%s\n", dest);
```
同样是上面这个例子，使用 `strcpy` 函数实现如下:
```c
char src[] = " World";
char dest[12] = "Hello";
strcpy(dest + strlen(dest), src);
printf("%s", dest);
```
效果也是一样的， `strcpy` 第一个参数为 `dest` 字符串的尾地址，然后将 `src` 字符串拷贝到这个地址。

### 参考资料 {id="also-resource"}

1. [维基百科-Ctype.h](https://zh.wikipedia.org/wiki/Ctype.h)
2. 慕课网-C语言系统化精讲