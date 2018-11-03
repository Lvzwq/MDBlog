---
layout: post
title: "Flask项目部署"
date: 2015-12-25 23:13:07 +0800
comments: true
categories: Dev
tags: [Python,Docker]
---

<!--more-->

## 使用`gunicorn`部署`Flask`项目
添加项目中使用到`python`第三方扩展，如果使用`virtualenv`的话，可以在虚拟环境中使用`pip freeze`命令直接导出。

```bash
pip freeze > requirements.txt
```

如下是所需要的扩展。

```
Flask==0.10.1
Jinja2==2.7.3
PyYAML==3.11
SQLAlchemy==1.0.4
Werkzeug==0.9.6
requests==2.2.1
ConfigParser==3.5.0b2
gunicorn==19.4.1
uWSGI==2.0.11.2
```
可以先在本地搭建一下服务。使用`docker`的话，其实就是把这个服务放在一个容器中，使用一个端口与本机交互。
```bash
pip install -r requirements.txt  #安装扩展
gunicorn -w 4 helper:app -b :8000  # 启动服务,服务端口为8000
```

`gunicorn`启动命令中, `-w 4`表示启动进程数量，`-b`表示绑定的主机和端口,默认是`127.0.0.1:8000`。详细的配置参考[http://docs.gunicorn.org/en/latest/settings.html](http://docs.gunicorn.org/en/latest/settings.html),
其中`helper:app`分别表示`helper.py`中的`app`应用

`helper.py`

```python
#!/usr/bin/env python
# coding: utf-8
from main import create_app
app = create_app()
app.debug = True
# app.run(host="0.0.0.0", port=5000)
```


其中`create_app`已经初始化`Flask`配置了。

### 使用`gunicorn`过程中的注意事项:

* 要将`app.run()`注释掉。
如果没有注释的话，会出现下面这种错误。

```
[2015-12-25 13:05:10 +0800] [51268] [INFO] Starting gunicorn 19.4.1
[2015-12-25 13:05:10 +0800] [51268] [INFO] Listening at: http://0.0.0.0:8004 (51268)
[2015-12-25 13:05:10 +0800] [51268] [INFO] Using worker: sync
[2015-12-25 13:05:10 +0800] [51271] [INFO] Booting worker with pid: 51271
 * Running on http://0.0.0.0:5000/
 * Restarting with reloader
[2015-12-25 13:05:11 +0800] [51272] [INFO] Starting gunicorn 19.4.1
[2015-12-25 13:05:11 +0800] [51272] [ERROR] Connection in use: ('', 8000)
[2015-12-25 13:05:11 +0800] [51272] [ERROR] Retrying in 1 second.
[2015-12-25 13:05:12 +0800] [51272] [ERROR] Connection in use: ('', 8000)
[2015-12-25 13:05:12 +0800] [51272] [ERROR] Retrying in 1 second.
[2015-12-25 13:05:13 +0800] [51272] [ERROR] Connection in use: ('', 8000)
[2015-12-25 13:05:13 +0800] [51272] [ERROR] Retrying in 1 second.
[2015-12-25 13:05:14 +0800] [51272] [ERROR] Connection in use: ('', 8000)
...
```

* 启动命令中使用`-b :8000`
如果使用

```
gunicorn -w 4 -b 127.0.0.1:8000 helper:app
```

在浏览器中就只能通过`localhost:8000`或者`127.0.0.1:8000`访问了，而不能直接通过`ip`访问。
应该设置为:

```
gunicorn -w 4 -b :8000 helper:app
或者
gunicorn -w 4 -b 0.0.0.0:8000 helper:app
```

现在开始将这个简单的项目部署到`docker`中,
首先看项目目录结构

```
.
├── README.md
├── config
│   ├── __init__.py
│   └── helper.conf
├── helper.py
├── library
│   ├── __init__.py
│   ├── curl.py
├── main
│   ├── __init__.py
│   ├── car.py
├── static
├── templates
└── uwsgi.ini
```
1. 添加`Dockerfile`文件
```
cd /path/to/project
touch Dockerfile

# 添加以下内容
FROM daocloud.io/python:2.7
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/requirements.txt
RUN pip install -r /code/requirements.txt
COPY . /code
COPY docker-entrypoint.sh docker-entrypoint.sh
RUN chmod +x docker-entrypoint.sh
EXPOSE 8000

CMD /code/docker-entrypoint.sh
```
项目服务启动的端口为8000，所以暴露8000端口，供主机访问

2. 添加启动文件`docker-entrypoint.sh`

```
#!/bin/sh
/usr/local/bin/gunicorn -w 4 -b :8000 helper:app
```

3. 将当前项目生成镜像

```
$ docker build -t helper .
```

4. 使用生成的镜像启动容器

```
$ docker run -p 8090:8000 helper
```

可以加上`-d`参数使项目在后台运行。可以直接在浏览器中通过
`http://主机ip:8090`访问项目了。之后可以直接设置`nginx`转发端口来绑定域名。


## 使用`uWSGI`来部署`Flask`项目

`Centos`上安装`uWSGI`

```
yum install -y python python-devel uwsgi-plugin-python
pip install uwsgi
```

直接命令行启动服务

```
uwsgi --http-socket 0.0.0.0:8000 --plugins python -w helper:app
或者
uwsgi --http-socket 0.0.0.0:8000 --plugins python --module helper --callable app --master --processes 4 --enable-threads --threads 4
```

### 使用`INI`配置文件, `helper.ini`如下

```
[uwsgi]
# uwsgi 启动时所使用的地址与端口
socket = 0.0.0.0:8000
# 或者
http-socket = 0.0.0.0:8000

# 指向网站目录
chdir = /root/helper

plugin = python

# python 启动程序文件
wsgi-file = helper.py

# python 程序内用以启动的 app 变量名
callable = app

# 处理器数
processes = 4

# 线程数
threads = 2
```

执行启动命令

```
uwsgi -i helper.ini
```


### 使用`XML`配置文件

```xml
<?xml version="1.0"?>
<uwsgi>
    <chdir>/root/helper</chdir>
    <!-- 项目对应的文件夹 -->
    <pythonpath>/root/helper</pythonpath> 
    <plugins>python</plugins>
    
    <!-- 对应helper.py -->
    <module>helper</module> 
    <callable>app</callable>  
    <master/>
    
    <socket>/tmp/helper.sock</socket>
    <!-- 或者使用 -->
    <!-- <http-socket>0.0.0.0:8000</http-socket> -->
    
    <!-- 使用虚拟环境-->
    <!-- <virtualenv>/root/helper/env/</virtualenv> -->
    <uid>root</uid>
    <gid>root</gid>
    <chmod-socket>644</chmod-socket>
    <chown-socket>root</chown-socket>
    <enable-threads>true</enable-threads>
    <processes>4</processes>
    <!-- 后台运行，打印日志 -->
    <daemonize>/root/helper/logs/%n.log</daemonize>
    <memory-report/>
</uwsgi>
```


执行启动命令

```
uwsgi -x helper.xml
```

### 使用`uWSGI`注意事项

1. 在`helper.py`中不要加`app.run()`
如果加上`app.run()`,会出现以下错误`unable to load configuration from uwsgi`


```
your mercy for graceful operations on workers is 60 seconds
mapped 363840 bytes (355 KB) for 4 cores
*** Operational MODE: preforking ***
added /root/helper/ to pythonpath.
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
unable to load configuration from uwsgi
```

另外，在`helper.py`中，在`if __name__ == '__main__':`中的内容不会加载。

2. 在配置内容中需要加上`uWSGI`对`Python`的支持

```
<plugins>python</plugins>
或者 ini文件中
plugins = python
```

没有加上，可能会出现

```
unable to load app 0 (mountpoint='') (callable not found or import error)
*** no app loaded. going in full dynamic mode ***
*** uWSGI is running in multiple interpreter mode ***
```
但是有的时候，也说可以不用加上，这个视情况而定。
还有一种情况，是出现`uWSGI`对`Python`的扩展没有安装，错误如下
```
[uWSGI] parsing config file uwsgi.xml
!!! UNABLE to load uWSGI plugin: dlopen(./python_plugin.so, 10): image not found !!!
```
在`Centos`执行
```
yum install -y uwsgi-plugin-python
```

3. [`invalid request block size: 21573 (max 4096)...skip`](http://stackoverflow.com/questions/15878176/uwsgi-invalid-request-block-size)
uWSGI日志如下
```
WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x1c32840 pid: 22461 (default app)
mountpoint  already configured. skip.
spawned uWSGI worker 1 (pid: 22461, cores: 2)
spawned uWSGI worker 2 (pid: 22466, cores: 2)
spawned uWSGI worker 3 (pid: 22467, cores: 2)
spawned uWSGI worker 4 (pid: 22468, cores: 2)
invalid request block size: 21573 (max 4096)...skip
invalid request block size: 21573 (max 4096)...skip
```
这个是因为在`uWSGI`配置文件中，设置了`socket`配置项为`0.0.0.0:8000`或者`socket`为`/tmp/uwsgi.sock`， 这样设置是不能通过浏览器访问的，还需要绑定`nginx`转发。可以将`socket`改为
```
<http-socket>0.0.0.0:8000</http-socket>
```

这样可以直接通过`IP:8000`直接访问。


## 使用`nginx`代理
设置的是`socket`的话，`nginx`配置如下
```
server {
        listen 80;
        server_name www.example.com;
        access_log /var/log/nginx/www.example.com/access.log;
        error_log /var/log/nginx/www.example.com/error.log;
        location / {
                include uwsgi_params;
                uwsgi_pass 0.0.0.0:8000;
		        # uwsgi_pass unix:/tmp/helper.sock;
        }
}
```
如果使用了`http-socket`的话，这种可以直接在浏览器中直接`IP`访问，可以直接设置代理。也可以把服务部署在`docker`中，使用上面的配置

```
server {
    listen 80;
    server_name www.example.com;
    access_log /var/log/nginx/www.example.com/access.log;
    error_log /var/log/nginx/www.example.com/error.log;
    
    error_page   500 502 503 504  /50x.html;

    location / {
        proxy_pass              http://localhost:8090;
        proxy_set_header        Host $host;
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_buffering off;
        add_header X-Accel-Buffering no;
    }
}
```