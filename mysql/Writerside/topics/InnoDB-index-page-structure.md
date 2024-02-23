# InnoDB索引页结构

在 InnoDB 中有设计了许多页，比如存放表空间头部信息的页、存放 Change Buffer 的页，而这篇文档描述的是存放数据的页，也就是索引页。

## 数据页的结构 {id="data-page-structure"}

数据页默认为 16KB，这 16KB 的存储空间划分为多个部分，如下图所示:

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/NaZvcvgURTDQspF7qUy5.png)

这 7 个部分如下表描述:

| 名称                 | 中文           | 描述          |
|--------------------|--------------|-------------|
| File Header        | 文件头部         | 页的一些通用信息    |
| Page Header        | 页头部          | 数据页专有的一些信息  |
| Infimum + Supremum | 页中的最小记录和最大记录 | 两个虚拟的记录     |
| User Records       | 用户记录         | 用户存储的记录内容   |
| Free Space         | 空闲空间         | 页中尚未使用的空间   |
| Page Directory     | 页目录          | 页中某些记录的相对位置 |
| File Trailer       | 文件尾部         | 校验页是否完整     |

## User Records 和 Free Space {id="user-records-and-free-space"}

当我们插入一条记录的时候，就会从 Free Space 中申请一块空间创建 User Records, 然后将插入的数据不断写入 User Records, 直到 Free Space 被用尽。

## Infimum 和 Supremum {id="infimum-and-supremum"}

这其实是 User Records 的一部分，在一个页中，在我们插入记录之前，InnoDB 会插入两条记录，分为了 Infimum 以及 Supremum。Infimum 表示页中
最小的记录，而 Supremum 表示页中最大的记录。这两条记录被称之为伪记录或虚拟记录。

![image.png](http://file-linker.oss-cn-hangzhou.aliyuncs.com/lYlxRnZ4RosBLWnb0H9V.png)

这些记录按照从小到大的顺序排列，区分大小通过记录的主键。并且规定，任何用户插入的记录都比 Infimum 记录大，且比 Supremum 记录小。

所以在记录的头信息中，这两条记录的`heap_no`分别是 0 和 1。我们自己插入的记录的 `heap_no` 从 2 开始。


