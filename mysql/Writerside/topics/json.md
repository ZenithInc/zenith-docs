# 使用JSON

MySQL 在 5.7.8 版本中提供了对原生 JSON 文档([RFC 7159](https://tools.ietf.org/html/rfc7159))的支持，到了 MySQL8.0 支持了 JSON Merge Patch 协议([RFC 7395](https://tools.ietf.org/html/rfc7396))。文本将介绍如何在 MySQL 中使用 JSON 数据格式。

## JSON 的存储与更新 {id="storage-and-update"}

在 MySQL 中，我们可以为字段指定数据类型为 JSON 格式。但是，需要注意的是，我们存入的数据为 JSON 的字符串，MySQL 会将其转换成内置的二进制格式存储。MySQL 会主动的校验我们存入的 JSON 文档的格式，如果格式不正确，会产生一个错误。

首先我们创建一个 JSON 的测试表:
```sql
create table if not exists json_test( json_str json not null );
```
> 注意: 在 MySQL 8.0.13 以及之前的版本，不能有非空的默认值。


插入一条 JSON 数据:
```sql
insert into json_test (json_str) value ('{"name": "JinZhi", "age": 14}');
```
> 注意: JSON 的最大存储长度和 LONGBLOB、LONGTEXT 是一致的，也就是必须小于 2 的 32 次方。它实际使用的存储空间是其长度加上4个字节。因此 LONGBLOB、LONGTEXT、JSON 都是变长的类型。


我们可以通过 `json_storage_size` 这个函数来查看 JSON 字符串的实际存储大小(对这个列更新之前的)：
```sql
select json_storage_size('{"name": "JinZhi", "age": 14}');
```
在 MySQL8.0 中，优化了对 JSON 局部更新的存储空间以及性能，并非简单的使用新的 JSON 文档来替换旧的 JSON 文档，但是这是有条件的。并非所有的情况都适用这种更新策略，比如说整个 JSON 文档都完全变了，以及下面的这些情况:

- 首先，更新的列必须定义为 JSON 的数据类型。
- 如果在更新语句中，使用了 `JSON_SET` 、 `JSON_REPLACE` 、 `JSON_REMOVE` 这些函数生成值的话，优化器并不会采用局部优化策略。
- 被更新的值不能小于更新值，比如原本是个整型，如果替换成了长整型，也不会采用更新策略。
- 其他的原因，可以查看[官方说明](https://dev.mysql.com/doc/refman/8.0/en/json.html)。

## JSON 相关的函数 {id="functions"}

MySQL 为 JSON 提供了一些函数，我们来了解一下。比如 `JSON_TYPE` 函数可以返回 JSON 的类型是数组还是对象:
```sql
> select json_type('["Abc", 1]');						-- ARRAY
> select json_type('{"Name": "JinZhi"}');		-- OBJECT
```
我们也可以使用另一个函数 `JSON_VALID` 来验证 JSON 字符串的格式是否正确:
```sql
> select json_valid('[1, 2]');   -- 1
-- 下面这两个例子说明了 json_valid 对于 null 的大小写是敏感的，使用的时候要小心
> select json_valid('NULL');   -- 0
> select json_valid('null');	-- 1
```

另外，我们还可以使用一些函数来构造出 JSON 字符串来，比如 `JSON_OBJECT` 和 `JSON_ARRAY` :
```sql
> select json_object('Name', "JinZhi", "Age", 18);  -- {"Age": 18, "Name": "JinZhi"}
> select json_array(1, 2, 3, 4, 5);  -- [1, 2, 3, 4, 5]
```
再来，介绍一个函数，它可以合并两个或更多的 JSON 字符串 -- `JSON_MERGE_PRESERVE` ：
```sql
-- [1, 3, 2, {"Name": "JinZhi"}]
> select json_merge_preserve('[1, 3, 2]', '{"Name": "JinZhi"}');
```

## JSON 路径表达式 {id="json-path-expression")

首先来介绍一下 JSON PATH 表达式中的一些符号:

| 符号       | 描述                      |
|----------|-------------------------|
| *        | 通配符                     |
| .        | 获取子对象                   |
| $        | 表示根路径                   |
| [n]      | 数组的下标                   |
| [n to m] | 对数组进行截取，从 n 个元素到第 m 个元素 |
| [last]   | 获取数组最后一个元素              |


如何对 JSON 字符串中的内容进行查询呢？以文档开头的创建的表和插入的数据为例：

```sql
> select * from json_test;  -- {"age": 14, "name": "JinZhi"}
```
如果要要查询年龄呢？使用 `->` 操作符就好了:
```sql
select json_str->"$.age" from json_test;
```
在插入一条更为复杂的文档:
```sql
insert into json_test (json_str) value 
	('{"Name": "JinZhi", "Hobbies": ["Read Book", "Play Game"]}');
```
我要查询我的第一项爱好呢？
```sql
> select json_str->"$.Hobbies[0]" from json_test;
+--------------------------+
| json_str->"$.Hobbies[0]" |
+--------------------------+
| NULL                     |
| "Read Book"              |
+--------------------------+
```
我要把值为 `NULL` 的记录过滤掉呢:
```sql
> select json_str->"$.Hobbies[0]" 
> from json_test 
> where json_str->"$.Hobbies[0]" is not null;
+--------------------------+
| json_str->"$.Hobbies[0]" |
+--------------------------+
| "Read Book"              |
+--------------------------+
```
从指定范围截取数组元素:
```sql
> select json_str->"$.Hobbies[last]" from json_test  -- Play Game
> select json_str->"$.Hobbies[0 to 1]" from json_test  -- ["Read Book", "Play Game"]
```