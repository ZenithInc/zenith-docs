# MySQL Dump

对于我们的应用来说，最重要的就是我们的数据。所以，如何保护好我们的数据成了每个项目开始之前，都应该做的计划。其中，备份更是尤为重要。可以让我们不丢失或者少丢失数据。

所以，这一篇文档，我们来描述在 MySQL 中，如何使用官方提供的 mysqldump 工具来对 MySQL 中的数据进行备份。

## 数据库备份分类 {id="categories"}

一般，数据库备份分为逻辑备份和物理备份。

逻辑备份的结果为 SQL 语句，适合用于所有的存储引擎。逻辑备份通常会要更多的时间，而 mysqldump 这样的工具就属于逻辑备份。

物理备份是对数据库目录的拷贝，对于内存表只备份表的结构。物理备份的恢复的数独通常比逻辑备份快。

物理备份分为在线备份和离线备份。如果要离线备份，需要对数据库服务进行停机，然后备份。如果使用在线备份则需要借助于第三方工具。

按照备份的内容划分，可以分为全量备份和增量备份。全量备份只值对所有的数据进行备份，而增量备份是指在全量备份之后，会数据库中更新的数据进行备份。

## 备份单表 {id="backup-a-table"}

如果是备份单表，可以使用 `insert...select` 语句, 下面是一个我在实际工作中的一个示例:
```SQL
INSERT into `orders_v3`
select `orders_v2`.*,
	case when `dts_source` = 'ring' or `dts_source` = 'dice' or `dts_source` = 'crash' or `dts_source` = 'sabang' or `dts_source` = 'color_game' or `dts_source` = 'scratch'
			then 'OG'
			else `dts_source`  
	end as `game_maker`
from `orders_v2`;
```

## MySQL Dump {id="mysql-dump"}

mysqldumnp 是 MySQL 官方提供的备份工具，使用比较简单。其基本语法如下：
```Shell
## 备份单一数据库
mysqldump [OPTIONS] database [tables]
## 备份多个数据库
mysqldump [OPTIONS] --database [OPTIONS] DB1 [DB2...]
## 备份 MySQL 实例下所有数据库
mysqldump [OPTIONS] --all-databases [OPTIONS]
```

其常用参数如下:

| 参数                   | 描述                                                                           |
|----------------------|------------------------------------------------------------------------------|
| -u                   | 指定用户名[1]                                                                     |
| -p                   | 指定密码                                                                         |
| --single-transaction | 对 InnoDB 引擎有效，启用事务                                                           |
| -l                   | 表示对所有的表进行锁定，期间只能对表进行读操作，对 InnoDB 引擎不需要使用这个参数                                 |
| -x                   | 锁定一个实例下的所有的表，数据库都会只读                                                         |
| --master-data        | 这个选项只有两个值，当值为 1 时备份文件中只记录了 CHANGE MASTER 语句，当值为 2 时，这语句会以注释的形式出现在备份文件中，默认为 1 |
| -R                   | 备份存储过程                                                                       |
| --triggers           | 备份触发器                                                                        |
| -E                   | 备份调度事件                                                                       |
| -tab                 | 表示生成两个备份文件，一个保存表结构，另一个保存数据                                                   |
| -w                   | 在到处单表的时候，可以使用该参数指定过滤条件                                                       |

> 这个账号需要具有如下权限: SELECT、RELOAD、LOCK TABLES、REPLICATION CLIENT、SHOW VIEW、PROCESS。

## 示例 {id="examples"}

在使用 mysqldump 之前，我们需要先建立一个备份专用的账号:
```Shell
create user 'backup'@'localhost' identified by 'your password';
```

为这个账号赋予权限:
```Shell
grant select,reload,lock tables,replication client,show view,event,process on *.* to 'backup'@'localhost';
## 刷新权限
flush privileges;
```

然后使用如下：
```Shell
mysqldump -ubackup -p --master-data=2 --single-transaction --routines --triggers --events shop > shop_backup.sql
```
> 注意: 需要开启 binlog 日志

恢复如下:
```Shell
mysql -uroot -p -e"create database shop;" ## 回车之后输入密码
mysql -uroot -p shop < shop_backup.sql. ## 回车之后输入密码
```

## 总结 {id="summary"}

这一篇文档我们介绍了数据库备份的分类，以及如何使用 MySQL Dump 这款工具来备份和恢复数据库。