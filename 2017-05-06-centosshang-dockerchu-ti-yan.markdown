---
layout: post
title: "Centos上Docker初体验"
date: 2015-12-24 23:10:21 +0800
comments: true
categories: Dev
tags: [Docker]
---


<!--more-->

Docker 依赖64位Centos系统。Centos 7内核至少为3.10.
通过`uname -r`查看内核版本，如果没有达到要求，可以参考[Centos7升级内核到3.18的方法](http://dl528888.blog.51cto.com/2382721/1609850)

```
# 导入key
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

# 安装elrepo的yum源
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

# 安装内核
yum --enablerepo=elrepo-kernel install  kernel-ml-devel kernel-ml -y

# 查看内核版本
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
# 修改默认启动顺序
grub2-set-default 0

# 重启
reboot
```


### Docker 入门
可以花几分钟看完这个[Docker 简介](http://dockerpool.com/static/books/docker_practice/introduction/what.html)， 算对Docker有一个初步的认识了。

先简要介绍Docker的几个指令。

`docker info`，查看docker 当前的运行状态


```
Containers: 1
Images: 4
Server Version: 1.9.1
Storage Driver: aufs
 Root Dir: /mnt/sda1/var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 6
 Dirperm1 Supported: true
Execution Driver: native-0.2
Logging Driver: json-file
Kernel Version: 4.1.13-boot2docker
Operating System: Boot2Docker 1.9.1 (TCL 6.4.1); master : cef800b - Fri Nov 20 19:33:59 UTC 2015
CPUs: 1
Total Memory: 996.2 MiB
Name: default
ID: KJEU:PWWQ:O7P2:M7YY:KGMN:CCO5:Q3SO:YXLC:LXWY:PQ2R:QXMQ:ABAL
.....
```

`docker` 命令

```
Options:

  --config=~/.docker              Location of client config files
  -D, --debug=false               Enable debug mode
  -H, --host=[]                   Daemon socket(s) to connect to
  -h, --help=false                Print usage
  -l, --log-level=info            Set the logging level
  --tls=false                     Use TLS; implied by --tlsverify
  --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA
  --tlscert=~/.docker/cert.pem    Path to TLS certificate file
  --tlskey=~/.docker/key.pem      Path to TLS key file
  --tlsverify=false               Use TLS and verify the remote
  -v, --version=false             Print version information and quit

Commands:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from a container to a HOSTDIR or to STDOUT
    create    Create a new container
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    exec      Run a command in a running container
    export    Export a container's filesystem as a tar archive
    history   Show the history of an image
    images    List images
    import    Import the contents from a tarball to create a filesystem image
    info      Display system-wide information
    inspect   Return low-level information on a container or image
    kill      Kill a running container
    load      Load an image from a tar archive or STDIN
    login     Register or log in to a Docker registry
    logout    Log out from a Docker registry
    logs      Fetch the logs of a container
    pause     Pause all processes within a container
    port      List port mappings or a specific mapping for the CONTAINER
    ps        List containers
    pull      Pull an image or a repository from a registry
    push      Push an image or a repository to a registry
    rename    Rename a container
    restart   Restart a running container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save an image(s) to a tar archive
    search    Search the Docker Hub for images
    start     Start one or more stopped containers
    stats     Display a live stream of container(s) resource usage statistics
    stop      Stop a running container
    tag       Tag an image into a repository
    top       Display the running processes of a container
    unpause   Unpause all processes within a container
    version   Show the Docker version information
    wait      Block until a container stops, then print its exit code

Run 'docker COMMAND --help' for more information on a command.
```

#### 创建容器
```
$ docker run -i -t ubuntu /bin/bash
Unable to find image 'ubuntu:latest' locally
14dab3d40372: Download complete
47d44cb6f252: Download complete
245bcdbcd071: Download complete
a57086046415: Download complete
8031fc57486d: Download complete
abe620e04d9d: Download complete
Status: Downloaded newer image for docker.io/ubuntu:latest
```

该命令会使用`Ubuntu`的镜像创建一个新的容器。`-i`表示Keep STDIN open even if not attached。`-t`表示Allocate a pseudo-TTY，它告诉Docker为要创建的容器分配一个伪tty终端。这样，新创建的容器才能提供一个交互式shell。

```
root@b874901e2f5b:/#
```

容器的主机名就是容器的ID。当前处于容器内`shell`状态，输入`exit`可以退出容器，退出容器后，容器也就停止了。

#### 查看容器和镜像

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker.io/centos    latest              14dab3d40372        5 days ago          194.7 MB
docker.io/ubuntu    latest              d55e68e6cc9c        12 days ago         187.9 MB

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
8e24480e37b9        centos              "/bin/bash"         5 minutes ago       Exited (0) 2 minutes ago                       distracted_aryabhata
b874901e2f5b        ubuntu              "/bin/bash"         33 hours ago        Exited (0) 7 minutes ago                       cocky_yonath
```
从该命令的输出结果中我们可以看到关于这个容器的很多有用信息：ID、用于创建该容器的镜像、容器最后执行的命令、创建时间以及容器的退出状态（在上面的例子中，退出状态是0，因为容器是通过正常的exit命令退出的）。我们还可以看到，每个容器都有一个名称。

`docker images` 会展示当前的所有基础镜像。`docker ps` 显示当前所有运行的容器，加上`-a`表示所有的容器。 

>有三种方式可以指代唯一容器：短UUID（如b874901e2f5b）、长UUID（如f7cbdac22a02e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778）或者名称（如cocky_yonath）。

`docker start containerID` 启动已经停止运行的容器
`docker stop containerID` 停止正在运行的容器
`docker pull ubuntu:latest`从远程仓库下载最新的Ubuntu版本镜像
`docker attach containID` 附着到容器上。如果是容器镜像是系统，将进入容器`Shell`模式。

### 查看容器内容运行情况
我们可以用 `docker logs containerID`获取容器内部运行的日志。
```
$ docker logs 9a0f7279429b
[23/Dec/2015 00:06:01]"GET /tree HTTP/1.1" 404 2083
[23/Dec/2015 00:06:03]"GET /favicon.ico HTTP/1.1" 404 2104
[23/Dec/2015 00:06:07]"GET / HTTP/1.1" 200 1176
[23/Dec/2015 00:06:07]"GET /static/main.css HTTP/1.1" 200 55
[23/Dec/2015 00:06:18]"GET / HTTP/1.1" 200 1176
[23/Dec/2015 00:06:56]"GET / HTTP/1.1" 200 1176
...
```
加上`-f`参数，监控`docker logs`日志。
```
docker logs -f 9a0f7279429b

docker logs --tail 10 9a0f7279429b 获取最后10行的日志
```

### 查看容器内的进程
```
docker top containerID
```

### 在容器内部运行程序
```
docker exec 57010f2568a2 mysql --version
docker exec 57010f2568a2 echo "hello world"
```

### 深入容器
我们可以用`docker inspect containerID`获取容器更多地信息。
`docker inspect`命令会对容器进行详细的检查，然后返回其配置信息，包括名称、命令、网络配置以及很多有用的数据。
```
$ docker inspect 4220b7df2d23
[
{
    "Id": "4220b7df2d23b23b3ecbab8ec86ad37b2f48d5830178b6353cd4fbbdab83bf58",
    "Created": "2015-12-19T19:03:36.639252716Z",
    "Path": "/bin/bash",
    "Args": [],
    "State": {
        "Status": "running",
        "Running": true,
        "Paused": false,
        "Restarting": false,
        "OOMKilled": false,
        "Dead": false,
        "Pid": 28778,
        "ExitCode": 0,
        "Error": "",
        "StartedAt": "2015-12-23T02:45:56.725884626Z",
        "FinishedAt": "2015-12-23T02:40:46.344221213Z"
    },
    "Image": "89d5d8e8bafb6e279fa70ea444260fa61cc7c5c7d93eff51002005c54a49c918",
    ...
```
有选择地获取`docker inspect`中的`docker`容器信息。`-f`有选择获取信息。
```
docker inspect --format '{{.State.FinishedAt}}' 4220b7df2d23
docker inspect -f '{{.Image}}' 4220b7df2d23
```

### 使用`docker-machine`来创建`Docker`(Mac && WIN)
`Docker`官方提供的部署工具。帮助用户快速在运行环境中创建虚拟机服务节点，在虚拟机中安装并配置`Docker`，最终帮助用户配置`Docker Client`，使得 `Docker Client` 有能力与虚拟机中的 Docker 建立通信。
```
docker-machine create -d virtualbox default
```
首先通过 `create` 命令创建一台名为 `default` 的 VirtualBox 虚拟机，并已经安装好了 `Docker`。
激活虚拟机`default`的环境变量。
```
 eval "$(docker-machine env default)"
```

`docker-machine` 命令行
```
─$ docker-machine --help
Usage: docker-machine [OPTIONS] COMMAND [arg...]

Create and manage machines running Docker.

Version: 0.5.4, build 6643d0e

Author:
  Docker Machine Contributors - <https://github.com/docker/machine>

Options:
  --debug, -D						Enable debug mode
  -s, --storage-path "/Users/ilovey/.docker/machine"	Configures storage path [$MACHINE_STORAGE_PATH]
  --tls-ca-cert 					CA to verify remotes against [$MACHINE_TLS_CA_CERT]
  --tls-ca-key 						Private key to generate certificates [$MACHINE_TLS_CA_KEY]
  --tls-client-cert 					Client cert to use for TLS [$MACHINE_TLS_CLIENT_CERT]
  --tls-client-key 					Private key used in client TLS auth [$MACHINE_TLS_CLIENT_KEY]
  --github-api-token 					Token to use for requests to the Github API [$MACHINE_GITHUB_API_TOKEN]
  --native-ssh						Use the native (Go-based) SSH implementation. [$MACHINE_NATIVE_SSH]
  --bugsnag-api-token 					BugSnag API token for crash reporting [$MACHINE_BUGSNAG_API_TOKEN]
  --help, -h						show help
  --version, -v						print the version

Commands:
  active		Print which machine is active
  config		Print the connection config for machine
  create		Create a machine
  env			Display the commands to set up the environment for the Docker client
  inspect		Inspect information about a machine
  ip			Get the IP address of a machine
  kill			Kill a machine
  ls			List machines
  regenerate-certs	Regenerate TLS Certificates for a machine
  restart		Restart a machine
  rm			Remove a machine
  ssh			Log into or run a command on a machine with SSH.
  scp			Copy files between machines
  start			Start a machine
  status		Get the status of a machine
  stop			Stop a machine
  upgrade		Upgrade a machine to the latest version of Docker
  url			Get the URL of a machine
  version		Show the Docker Machine version or a machine docker version
  help			Shows a list of commands or help for one command

Run 'docker-machine COMMAND --help' for more information on a command.
```
#### 查看当前正在运行的`Machine`
```
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER   ERRORS
default   *        virtualbox   Running   tcp://192.168.99.100:2376           v1.9.1
```
#### 启动虚拟机`Machine`
```
docker-machine start default
```
#### 获取虚拟机`Machine`的`IP`
```
docker-machine ip default
```
#### 通过`SSH`进入到虚拟机
```
$ docker-machine ssh default
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.9.1, build master : cef800b - Fri Nov 20 19:33:59 UTC 2015
Docker version 1.9.1, build a34a1d5
docker@default:~$
```
#### 参考
[Centos7升级内核到3.18的方法](http://dl528888.blog.51cto.com/2382721/1609850)
[迈出使用Docker的第一步，学习第一个Docker容器](http://www.jianshu.com/p/26f15063de7d)
