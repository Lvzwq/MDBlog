---
layout: post
title: "SQLAlchemy(一)"
date: 2015-10-31 17:45:47 +0800
comments: true
categories: Dev
tags: [Python, MySQL]
---

<!--more-->
### 使用`create_engine`连接数据库
```
from sqlalchemy import create_engine
conn = "mysql://root:root@localhost/dbname?charset=utf8"
engine = create_engine(conn, echo=True) # 在console中输出SQL语句
```
`create_engine`方法返回的是一个`Engine`对象实例，`Engine`对象实例`engine`并没有立即连接数据库，只有在第一次有与数据库操作的任务时才会去连接数据库。

### 定义和创建表
```
from sqlalchemy import Table, Column, Integer, String, MetaData, ForeignKey
metadata = MetaData()
```
`SQLAlchemy` 使用 `ORM` 来映射对象和表的关系。
表(`Table`)的集合以及与它们有关系的子类(`child object`)  称为`数据库元数据`(`database metadata`)


```
users = Table('user', metadata,
        Column('id', Integer, primary_key=True),
        Column('name', String(32), nullable=False),
        Column('gender', String(1), nullable=False, server_default='M')
        )
metadata.create_all(engine)
```
创建好表对象之后，我们来创建表。
使用`metadata`的`create_all`方法来在数据库中创建表。`create_all`这个方法首先会在数据库中查找对应的表是否存在，存在则不创建，所以这个方法多次调用是安全的。

### 数据库操作
* 插入
```python
# Insert 构造对应insert语句
ins = users.insert() 
# <sqlalchemy.sql.dml.Insert object at 0x1079a6290>
print str(ins) 
# INSERT INTO "user" (id, name, gender) VALUES (:id, :name, :gender)
ins = users.insert().values(name='jack', gender='M')
# 也可以这样

```
执行插入操作
我们创建的`engine`对象是一个能执行SQL语句的数据库仓库。我们使用`connect`来创建一个连接.
```
conn = engine.connect()
# 执行插入操作
result = conn.execute(ins) # 执行SQL插入操作,返回一个ResultProxy object
print result.rowcount # 显示插入条数
print result.inserted_primary_key # 显示插入的id
```
* 批量插入
```
conn.execute(users.insert(), [
    {'name' : 'Zs', 'gender': 'F'},
    {'name' : 'Ls', 'gender': 'M'},
    {'name' : 'Ww', 'gender': 'F'}
    ])
```

* 查询
主要的创建`select`的SQL语句的方法是`select()`
```
from sqlalchemy import select
s = select([users]) # select参数是一个列表  
r = conn.execute(s) # 返回的是一个ResultProxy object
all = conn.fetchall() # 返回一个含有所有RowProxy的列表
print all[0].name, all[0].id
print all[0]['name'], all[0]['id']
print all[0][0], all[1][1] # RowProxy 不支持赋值操作,可以强制转换为dict
```
条件查询
```
s = select(['user', 'address']).where(user.c.id == address.c.user_id)
```
连接词
```python
from sqlalchemy import and_,  or_, not_
c = and_(users.c.name.like('j%'), users.c.gender == 'M', not_(users.c.id == 1)
s = select([users.c.id, users.c.name]).where(c)
```
使用原生SQL语句
```
from sqlalchemy import text
s = text（"SELECT * FROM users WHERE users.gender = :gender and name = :name")
result = conn.execute(s, gender='F', name='Z').fetchall()

# 也可以在select()中使用text语句
s = select([users.c.name]).where(text("users.id = 2"))
result = conn.execute(s).fetchall()
# 尽量少使用text
```
使用假名
```
u1 = users.alias()
u2 = users.alias()
```
使用连接
```
print users.join(address) # 需要之前设置外键
print users.join(address, users.c.id == address.c.user_id)

s = select([users.c.name, address.c.email]).select_from(users.join(address, address.c.user_id == users.c.id)).where(address.c.email.like("%qq.com"))

# 产生的SQL语句为:
`SELECT "user".name, address.email \nFROM "user" JOIN address ON address.user_id = "user".id \nWHERE address.email LIKE :email_1`
```
* 使用SQLalchemy调用数据库的`function`函数
```
from sqlalchemy.sql import func
s = select([func.max(users.c.id).label("max_id")]).where(user.c.name.like("Z%))
```
* 排序
```
s = select([users]).order_by(users.c.id)
s = select([users]).order_by(users.c.id.desc())
```
* 其他操作
```
s = select([users]).having(func.length(users.c.name) > 4)
s = select([users], func.sum(users.c.score).lable("score_all")).group_by(users.c.gender)
```

### 更新和删除
```
up = users.update().where(users.c.id == 1).values(name="testname")

up = users.update().where(users.c.id == 1).values(name="prefix_" + users.c.name)
# 使用bindparam来操作更新数据
from sqlalchemy import bindparam
stmt = users.update().where(users.c.id == bindparam("user_id")).values(name = bindparam("new_name"))
s = conn.execute(stmt, user_id=1, new_name="hello")

批量更新数据
s = conn.execute(stmt, [
    {'user_id': 1, 'new_name': 'good'},
    {'user_id': 2, 'new_name': 'bad'}
    ])
```

删除数据
```
d = users.delete()
conn.execute(d) # 删除全部数据

d = users.delete().where(users.c.gender == "M")
```