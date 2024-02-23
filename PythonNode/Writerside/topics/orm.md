# 使用 ORM 操作数据库

这一篇文档，我们来讲讲数据库。因为我们现在的应用大部分都是面向数据库的，所以说写一个项目基本上也都围绕着数据库展开的。因此掌握数据库的操作便是重中之重。

当前，ORM 思想十分流行。在 Python 社区，ORM 主要指的就是 SQLAlchemy, 而 Flask 框架的插件 Flask-SQLAlchemy 就是基于 SQLAlchemy 的。所以，这一篇文档，我们也是主要描述 SQLAlchemy 的应用。

## 安装依赖 {id="install"}

首先需要先安装下面这些依赖，不然使用 pip 安装 Python 依赖的时候会报错, 下面以 CentOS/RedHat 发行版为准，其他发行版类似:

```Shell
sudo yum install gcc python3-devel mysql-devel
```

然后安装 Python 数据库相关依赖:

```Shell
python3 -m pip install mysqlclient
python3 -m pip install Flask-SQLAlchemy
```

Flask-SQLAlchemy 依赖于SQLAlchemy， 而 SQL Alchemy 依赖与 PyMySQL。

## 什么是 ORM {id="what"}

ORM(Object Relational Mapping), 字面意思就是对象关系的映射。这对于刚接触 ORM 的同学来说，可能并不好理解。

简单来说，就是用面向对象中的对象去对应数据库中的表，用属性去表示数据库中的字段，每一个条数据库中的记录就是一个对象，每一个字段就是一个对象属性。

比如下面这个例子，User 表对应着代码中的 User 对象。User 表中有三个字段，分别是 id、username 以及 对象也创建了这三个属性。

```Python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)
```

实际上，我们的程序仍旧是将对象最终转成 SQL 去查询数据库的。之所以不使用 SQL 去查询数据库，而将数据表转成对象，看上去是多此一举，但这会让我们的代码更加的简洁、可读以及便于管理。

是否使用 ORM，或是使用 SQL 直接查询数据库，这在业界是有争议的。实际上，我觉得 ORM 降低了数据库相关操作在代码中的使用难度，其实可以给出一种比较优雅的写法，这也是很多人喜欢使用 ORM 的原因。但这并不意味，不使用 ORM 就不能写出优雅的代码，只是对开发者要求更高而已。

从上面的示例代码我们可以看到，Integer 对应的是数据库中的 Int 类型，而 String 对应的是数据库中的 Varchar 类型。试想一个问题，MySQL 当中的 TinyInt 如何表示呢？答案是使用 Boolean。为什么呢？

因为 MySQL 中的 TinyInt 类型不是标准的 SQL 数据类型，所以 SQLAlchemy 使用了 Boolean 来映射不同数据库中的不同的数值类型。比如在 MySQL 中，Boolean 会转为 TinyInt；而在 Postgresql 数据库中，Boolean 会转成 Boolean；在 SQL Server 中会转成 Bit。

## CURD 示例 {id="curd"}

下面是增删改查的基本示例。

### 定义模型 {id="defined"}

编写入口文件如下:
```Python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://test:Xbcd20198$@127.0.0.1/test'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)

    def __init__(self, username, email):
        self.username = username
        self.email = email

    def __repr__(self):
        return '<User %r>' % self.username
```

然后使用 Python 交互式命令行，在数据库中创建上面代码中定义的表:

```Shell
$ python3
>> from app improt db
>> db.create_all()
```

这个之后，我们登录数据库服务器，就可以看到，数据库中已经存在 user 表了。

```Shell
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| user           |
+----------------+
```

### 添加数据 {id="create"}

然后我们借助 Python3 提供的交互式 Shell 来添加一些数据:
```Shell
$ python3
>>> from app import User, db
>>> admin = User('admin', 'admin@end.wiki')
>>> guest = User('guest', 'guest@end.wiki')
>>> db.session.add(admin)
>>> db.session.add(guest)
>>> db.session.commit()
```

### 查询数据 {id="query"}

如果要查询一张表的记录数:
```Shell
>>> from app import db, User
>>> db.session.Query(User).count()
5
```

如果要查询全部数据:
```Shell
>>> from app import db, User
>>> db.session.Query(User).all()
```

如果要查询全部数据:
```Shell
>>> from app import db, User
>>> db.session.Query(User).filter(User.name == 'bob').first()  # 查询符合条件的第一条
>>> db.session.Query(User).filter(User.name == 'bob').all()    # 查询符合条件的全部记录
# 如果是多个条件
>>> db.session.Query(User).filter(User.name == 'bob', User.id > 2).all()
```

<note>
对于 `...filter(User.name == 'bob')...` 这样的写法不要感到奇怪，不应该传入一个布尔值嘛？传入布尔值如何去过滤呢？这里使用了运算符重载，这里返回的是对象。你可以运行 `type(User.name == 'bob')`看看返回的是什么？应该是 `'sqlalchemy.sql.elements.BinaryExpression'>`。
</note>


### 删除数据

删除数据就是在查询的基础上，使用 `delete` 方法:
```Shell
>>> from app import db, User
>>> db.session.Query(User).filter(User.name == 'bob').delete()  # 查询符合条件的第一条
>>> db.session.commit()
```
