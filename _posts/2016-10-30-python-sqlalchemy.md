---
layout: post
title: Python使用SQLAlchemy连接并操作数据库
---

# Python使用SQLAlchemy连接并操作数据库

## 安装

### 安装 SQLAlchemy

使用 pip 工具直接安装 SQLAlchemy 库，这个库支持 Python 2.6 以上版本以及 Python 3.x 系列版本。

```
pip install SQLAlchemy
```

检查是否安装成功以及库版本，这里我们使用的是 1.1.3 版本。

```
>>> import sqlalchemy
>>> sqlalchemy.__version__
'1.2.2'
```

### 安装 Dialect

Dialect 是一个库，作用是把 SQLAlchemy 使用的通用SQL转换成各个数据库使用的SQL命令。

这篇文章里我们使用 SQLite 数据库，可以不用 Dialect，因为 SQLite 是内置在 Python 中的。如果你使用 MySQL 数据库，可以使用 PyMySQL 等 Dialect。具体各个数据库使用什么 Dialect，可以参考[官网教程]。

## 连接数据库

在连接数据库之前，我们要做一些准备工作，如输入数据库账号和密码，创建与表对应的类等操作，以下简单介绍一下要做什么，这些工作有什么用。

### 创建基类

我们首先要创建一些连接数据库所必须的类。

```
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

engine = create_engine("sqlite:///:memory:")

Base = declarative_base()

Session = sessionmaker(bind=engine)
```

代码首先从 SQLAlchemy 库中导入`create_engine`函数，这个函数返回一个 Engine 类，Engine 类保存了数据库的基本信息，如账号，密码等，并负责处理与数据库的连接。

`create_engine`需要一个包含账号密码的字符串作为参数，在这里我们使用的 SQLite 数据库不需要账号密码，只要输入数据库文件地址就可以了，`:memory:`意味着数据库存在于内存中。

各个数据库的参数要如何写，可以参考[官网教程database-urls]。

之后从 SQLAlchemy 库中导入一个`declarative_base`函数，这个函数返回一个类，这里是`Base`，我们之后创建的所有表类都是这个`Base`类的后代。`Base`类保存了所有已定义表的元信息。

最后是`sessionmaker`函数，这个函数返回一个工厂函数，我们把返回的工厂函数保存在`Session`里。每次调用这个工厂函数都会获得一个类，我们通过这个类对数据库进行操作。

`Session`类除了与数据库沟通外，还是一个保存区，保存了你对数据库的操作，你可以把多个操作加入到会话中，再一次性执行，还可以撤销你的操作等。

### 定义表类

接下来是定义表类，这些类每一个对应数据库中的一个表，我们通过新增和修改这些类来操作表数据。

```
from sqlalchemy import Column, Integer, String

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    password = Column(String)

    def __repr__(self):
        return "<User(name='{}', fullname='{}', password='{}')>".format(
            self.name,
            self.fullname,
            self.password
        )
```

上面的代码十分简单，目的是建立一个与表 `users` 对应的类`User`，之后可以通过 `User` 类来查询和修改表。

`User` 类的`__tablename__`指定了这个类对应的是数据库中的哪个表，而其他的属性都是一个`Column`类，对应表的一个列，需要的参数是每个列的类型和约束等。

`User.__table__`记录了表的元信息。

### 连接数据库

所有准备工作完成，之后就是连接到数据库了。

```
Base.metadata.create_all(engine)

session = Session()  # 创建会话
```

`Base`类的`metadata`属性保存了所有表的元信息，`create_all`函数检查每个表是否存在并创建不存在的表，然后连接到数据库。

之后创建一个`Session`类实例，之后就可以通过这个`session`与数据库沟通了。

## 数据库操作

### 新增行

要在数据库中新增一行数据，我们只要新建一个对应的类，然后把加入到会话中既可以了。

```
>>> u1 = User(name="ed", fullname="Ed Jones", password="fxfx012")
>>> session.add(u1)
>>>
>>> session.new
IdentitySet([<User(name='ed', fullname='Ed Jones', password='fxfx012')>])
>>>
>>> session.commit()
```

我们在这里创建一个新的`User`类，类中保存了我们要加入数据库的数据，再把 `u1` 通过 `session` 的 `add()` 方法加入到 `session` 中，再用 `session.commit()` 方法提交到数据库并执行。

在提交之前我们可以通过 `session.new` 属性查看我们将要新建的数据行。

我们还可以用 `session.add_all()` 方法一次性把多个新数据加入到 `session` 中。

```
>>> u2 = User(name="wendy", fullname="Wendy Williams", password="footer")
>>> u3 = User(name="mary", fullname="Mary Contrary", password="xxg527")
>>> session.add_all([u2, u3])
>>> session.commit()
```

### 查询行

要查询数据库，可以使用 `session.query()` 方法，`query`需要一个参数来指明要查询的是哪个表，参数值是类对象，`query`返回一个可迭代对象包含所有数据行，以类实例表示。

```
>>> data = session.query(User)
>>> for instance in data:
...     print(instance.name, instance.fullname)
...
ed Ed Jones
wendy Wendy Williams
mary Mary Contrary
```

把 `User` 类传入`query`函数指明要查询的是数据库中的 users 表。

```
>>> u = session.query(User).first()
>>> u
<User(name='ed', fullname='Ed Jones', password='fxfx012')>
>>>
>>> us = session.query(User).all()
>>> us
[<User(name='ed', fullname='Ed Jones', password='fxfx012')>,
 <User(name="wendy", fullname="Wendy Williams", password="footer")>,
 <User(name="mary", fullname="Mary Contrary", password="xxg527")>]
```

`query`返回的对象还可以继续进行选择, 如 `first()` 直接返回查询到的第一个数据，`all()` 返回一个列表。

使用 `order_by()` 方法可以对查询数据进行排序。

```
>>> us = session.query(User).order_by(User.name).all()
>>> us
[<User(name='ed', fullname='Ed Jones', password='fxfx012')>,
 <User(name="mary", fullname="Mary Contrary", password="xxg527")>,
 <User(name="wendy", fullname="Wendy Williams", password="footer")>]
```

`filter()`方法可以对数据进行过滤。

```
>>> us = session.query(User).filter(User.name == 'ed').all()
>>> us
[<User(name='ed', fullname='Ed Jones', password='fxfx012')>]
>>>
>>> us = session.query(User).filter(User.name.in_(["ed", "wendy"])).all()
>>> us
[<User(name='ed', fullname='Ed Jones', password='fxfx012')>,
 <User(name="wendy", fullname="Wendy Williams", password="footer")>]
```

`filter()`的参数可以使用 `User.name.in_()` 在一个范围内进行匹配。`filter()` 除了 `==` 和 `in_` 外还有其他条件可以使用，具体可以看[官网教程过滤操作]。

### 修改行

要修改数据，只要修改查询到的类的对应属性就可以了，`session` 会自动把修改加入要提交的命令中。

```
>>> u = session.query(User).first()
>>> u
<User(name='ed', fullname='Ed Jones', password='fxfx012')>
>>>
>>> u.password = "ggxx213"
>>> session.dirty
IdentitySet([<User(name='ed', fullname='Ed Jones', password='ggxx213')>])
>>>
>>> session.commit()
>>> u_change = session.query(User).first()
>>> u_change
<User(name='ed', fullname='Ed Jones', password='ggxx213')>
```

可以看到，但修改了 `u1` 的属性后，`session` 里自动添加了一项修改，可以用 `session.dirty` 查看将要提交的修改。

### 删除行

使用 `session.delete()` 删除一个数据行。

```
>>> u1 = session.query(User).first()
>>> u1
<User(name='ed', fullname='Ed Jones', password='ggxx213')>
>>>
>>> session.delete(u1)
>>> session.commit()
>>>
>>> session.query(User).all()
[<User(name="wendy", fullname="Wendy Williams", password="footer")>,
 <User(name="mary", fullname="Mary Contrary", password="xxg527")>]
```

### 回滚

我们可以撤销我们对数据库的上一次提交的所有操作，使用 `session.rollback()` 方法。

```
>>> u = session.query(User).first()
>>> u
<User(name='wendy', fullname='Wendy Williams', password='footer')>
>>> u.password = "header"
>>> fake_user = User(name="fakeuser", fullname="Invalid", password="12345")
>>> session.add(fake_user)
>>>
>>> session.dirty
IdentitySet([<User(name='wendy', fullname='Wendy Williams', password='header')>])
>>> session.new
IdentitySet([<User(name="fakeuser", fullname="Invalid", password="12345")>])
>>> session.query(User).all()
[<User(name="wendy", fullname="Wendy Williams", password="header")>,
 <User(name="mary", fullname="Mary Contrary", password="xxg527")>,
 <User(name="fakeuser", fullname="Invalid", password="12345")>]
>>>
>>> session.rollback()
>>> session.query(User).all()
[<User(name="wendy", fullname="Wendy Williams", password="footer")>,
 <User(name="mary", fullname="Mary Contrary", password="xxg527")>]
```

注意：`rollback()` 方法是撤销上一次的提交，`commit()` 是一次提交，而 `query()` 也是一次提交，因为它会在查询之前提交一次。

## 参考资料

SQLAlchemy 官网: <a href="http://www.sqlalchemy.org/">http://www.sqlalchemy.org/</a>

SQLAlchemy 文档: <a href="http://docs.sqlalchemy.org/en/latest/">http://docs.sqlalchemy.org/en/latest/</a>

SQLAlchemy 官网教程: <a href="http://docs.sqlalchemy.org/en/latest/orm/tutorial.html">http://docs.sqlalchemy.org/en/latest/orm/tutorial.html</a>

<!---------------- 链接 ---------------------->

[官网教程]: http://docs.sqlalchemy.org/en/latest/dialects/index.html

[官网教程database-urls]:  http://docs.sqlalchemy.org/en/latest/core/engines.html#database-urls

[官网教程过滤操作]: http://docs.sqlalchemy.org/en/latest/orm/tutorial.html#common-filter-operators
