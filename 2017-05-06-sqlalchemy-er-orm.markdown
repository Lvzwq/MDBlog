---
layout: post
title: "SQLAlchemy(二) --- ORM"
date: 2015-10-31 19:34:33 +0800
comments: true
categories: Dev
tags: [Python, MySQL]
---

<!--more-->

`SQLAlchemy ORM`提供了一个连接数据库表和用户自定义`Python`类的方法。

### 定义一个映射(Mapping)
类的映射使用已经在基类中定义的声明式系统,这个基类(`base class`)维护了一个和这个基类相关的数据表的对应关系, 这就是所谓的`declarative base class`。
我们的应用在一般在导入的模块中将只有一个这个基类的实例。我们使用`declarative_base()`创建这个基类的实例。
```python
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()
```
有了这个基类我们可以定义任何数量的映射类。

```python
from sqlalchemy import Column, Integer, String

class User(Base):
    __tablename__ = 'users'   
    id = Column(Integer, primary_key=True)
    name = Column(String(32), nullable=False)
    gender = Column(String(1), nullable=False, server_default='M')
    
    def __repr__(self):
        return "<User(name='%s', gender='%s')" % (self.name,                            self.gender)
        
# 执行创建表的语句
Base.metadata.create_all(engine)
```

一个继承自`Base`的类至少有一个`__tablename__`属性，并且至少有一个含有主键(`primary key`)的`Column`。当一个类创建时， `Declarative` 将所有的`Column`类用特殊的Python属性访问器替代。

### 创建Session
ORM操作数据库的句柄就是`Session`.当我们启动我们的应用时，在`create_engine()`的同时，我们定义了一个`Session`类将作为一个创建`Session`实例的工厂。

```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
```

如果我们还没有`Engine`的实例，我们可以先创建Session，当创建Engine后再绑定。
```
Session = sessionmaker()
Session.configure(bind=engine)
```

当需要和数据库有一个会话时，可以初始化一个`Session`

```
session = Session()
```

### 添加新的类
为了持久化`User`对象， 我们使用`add()`将它加入到`Session`中。

```
user = User(name="zhangsan", gender="M")
session.add(user)
```

此时，这个`user`实例是待定的(`pending`);不执行任何SQL也不代表数据库中的一行数据。当使用一个`flush`过程时，`Session`将执行SQL来持久化`user`。

```
our_user = session.query(User).filter_by(name='zhangsan').first()
# 此时our_user 和 user是同一个对象
```

事实上，`Session`返回同一行(对象)就是我们刚刚在`Session`内部的类的字典持久化的对象，所以我们事实上获得的就是我们加入`Session`中的实例。
        
我们需要告诉`Session`我们想要将所有的改变存入到数据库，提交事务。我们通过`commit()`来执行。

```
session.commit()
```

在执行`commit`之前，执行`query`后，即使`user`已经有了标识符`id`, 但是数据库中并没有提交数据，只有在`commit`之后才会提交数据。

我们可以在数据提交`commit`之前调用事务的`rollback`来回滚之前的修改。

```
session.rollback()
```

### 查询`Query`
`Query`查询返回的是元组`tuples`,是`KeyedTuple`提供的类，并且更像一个原生的Python对象。

```
for row in session.query(User, User.name).all(): 
    print row.User, row.name
```

`filter()`方法通常接收的是Python操作符，而`filter_by()`方法使用关键字参数

```
session.query(User).filter(User.name == "ed")
session.query(User).filter_by(name = "ed")
```

通用的查询语句
equals:

```
query.filter(User.name == 'ed')
```

not equals:

```
query.filter(User.name != 'ed')
```

LIKE:

```
query.filter(User.name.like('%ed%'))
```

IN:

```
query.filter(User.name.in_(['ed', 'wendy', 'jack']))

# works with query objects too:
query.filter(User.name.in_(
        session.query(User.name).filter(User.name.like('%ed%'))
))
```

NOT IN:

```
query.filter(~User.name.in_(['ed', 'wendy', 'jack']))
```

IS NULL:

```
query.filter(User.name == None)

# alternatively, if pep8/linters are a concern
query.filter(User.name.is_(None))
```

IS NOT NULL:

```
query.filter(User.name != None)

# alternatively, if pep8/linters are a concern

query.filter(User.name.isnot(None))
```

AND:

```
# use and_()
from sqlalchemy import and_
query.filter(and_(User.name == 'ed', User.fullname == 'Ed Jones'))

# or send multiple expressions to .filter()
query.filter(User.name == 'ed', User.fullname == 'Ed Jones')

# or chain multiple filter()/filter_by() calls
query.filter(User.name == 'ed').filter(User.fullname == 'Ed Jones')
```

OR:

```
from sqlalchemy import or_
query.filter(or_(User.name == 'ed', User.name == 'wendy'))
MATCH:

query.filter(User.name.match('wendy'))
```

### 返回列表`List`和标量`Scalar`

`all()`返回一个列表

```
query = session.query(User).filter(User.name.like('%ed')).order_by(User.id)
query.all()
```

`first()`使用限制，返回结果集的第一条作为`Scalar`

```
query.first()
```

`one()` 如果返回结果集不止一个对象，将`raise`一个错误。

```python
from sqlalchemy.orm.exc import MultipleResultsFound
try:
    users = query.one()
except MultiResultsFound e:
    print e
```

没有数据返回时

```
from sqlalchemy.orm.exc import NoResultFound
try:
    users = query.one()
except NoResultFound, e:
    print e
```

`scalar()`调用`one()`方法，并且成功时返回第一列数据

```
query.scalar()
```

### 使用文本SQL语句
文本字符串可以在查询中灵活使用，通过`text()`构建。

```
from sqlalchemy import text
for user in session.query(User).filter(text("id>20")).order_by(text("id")).all()
    print user.name
```

可以绑定参数，使用`params()`方法

```
session.query(User).filter(text("id<:value and name=:name")).params(value=220, name="ed").order_by(User.id).one()
```

计数方法`count()`

```
session.query(User).filter(Usre.name.like("%ed")).count()
```

也可以使用`func.count()`来计数

```
from sqlalchemy import func
session.query(func.count(User.name), User.name).group_by(User.name).all()
```
