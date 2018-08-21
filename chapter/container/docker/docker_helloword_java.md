# 第一个docker化的java应用

> 慕课免费课程：https://www.imooc.com/learn/824
> Gitbook 《docker从入门到实践》 https://yeasy.gitbooks.io/docker_practice/underly/ufs.html

简介：Docker是一个使用Go语言开发的开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的机器上。Docker的发展速度和火爆程度着实令人惊叹，一发不可收拾，形成了席卷整个IT界的新浪潮。学完本课程你将了解到什么是docker，docker的思想以及诸如镜像，仓库，容器等核心概念。你将学会怎样运行一个容器，如何搭建私有仓库，怎么写dockerfile以及怎样把自己的应用放到容器中运行。docker将会是你的IT路上一笔不小的财富。

目录：
* 第1章 课程介绍

  对docker有个简单的印象，了解课程的安排。
* 第2章 了解docker

  用形象的类比说明docker的集装箱、标准化、隔离的思想。在用几个工作学习中碰到的问题说明docker解决了哪些问题。
* 第3章 走进docker

  结合上面的类比引出docker的核心技术：镜像、仓库和容器的概念，并分别深入讲解技术、原理。

* 第4章 docker安装

  分别在三中平台上讲解docker的安装。同学可以选择自己的平台观看。

* 第5章 docker初体验

  第一个实例：用helloworld镜像带入，熟悉docker最基本的两个命令，拉取镜像和运行容器，并讲解背后运行逻辑。

* 第6章 docker运行nginx静态网站

  第二个实例：从运行nginx镜像引出docker网络概念和docker的端口映射，最后运行nginx容器。

* 第7章 第一个java web应用

  最后一个实例：创建自己的镜像，引出dockerfile，讲解基本的dockerfile语法。然后讲解私有仓库的搭建。最后分别在两台机器上演示docker的跨平台运行我们的java web项目。

* 第8章 课程总结


## 什么是docker？

历史：

* 2010 dotCloud PAAS ：竞争太大
* 2013 docker开源 ：发展不理想，决定开源
* 2014.6 Docker1.0
* 2014.7 C轮 $4000万
* 2015.7 D轮 $9500万
* 至今 Docker 1.13

 装应用的容器，开源把任何程序放进docker里；

## docker 思想
![](/assets/image/container/docker/docker_icon.jpg)

logo鲸鱼装货物，很整齐，能装很多；鲸鱼运输集装箱到超级码头，再分发

* 集装箱 ：如把程序所有相关配置都打包，解决在一个环境能运行，另外一个环境运行出错
* 标准化
  - 运输方式  ： 哪里需要则需要由小鲸鱼运输，如：把应用使用qq发送，u盘拷贝到目的地
  - 存储方式  ： 把程序copy到笔记本上，需要记住这个目录。不需要关心存储什么地方，使用命令执行即可
  - API接口  ： 接口标准化，如tomcat，nginx都有自己的启动方式，使用docker就一种启动方式
* 隔离 ：如虚拟机，而docker更轻量化，使用linux rxc技术，达到快速创建销毁，可能1秒就创建好了

## docker解决了什么问题？

* 我本地运行没问题！

    比如一个java web程序要启动起来需要依赖什么？
    - 操作系统
    - jdk版本
    - tomcat版本
    - 配置文件，或则与系统相关的。
    只要任意一个出现问题，都有可能运行不起来或则运行结果不对

    那么docker就是解决这个问题的，把各个配置打包成集装箱，又小鲸鱼送到服务器上，然后运行；
* 系统好卡，哪个哥们又写死循环了？

  公用服务器的时候会出现这种情况，某一个应用占用太多资源，导致另外一个应用崩溃；
  docker则的隔离机制则会在启动时就限制好了，你这个应用可用最大资源是固定的。不会影响到其他的应用
* 双11来了，服务器撑不住啦！

  热点事件的时候，访问流量巨大，平时部署这么多机器就纯粹的浪费人力物力；
  那么在热点事件来临之际，就需要运维开启大量的机器来支撑。
  docker就是解决这个问题的，只需要动动鼠标，配置配置即可快速扩容
## 走进docker

核心词汇：
* 镜像 ：集装箱
* 仓库 ：超级码头
* 容器 ：程序运行的地方

去仓库把镜像拉倒本地，用命令把程序运行起来，变成服务器；

Docker - Build, Ship, and Run Any App, Anywhere

* build ： 构建 镜像
* ship ： 运输 镜像
* run ： 运行 镜像

## docker镜像

image ：一系列的文件；文件就会有存储到哪里?格式是什么？

> 引用 ： https://yeasy.gitbooks.io/docker_practice/underly/ufs.html

联合文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。

简单说：将不同的目录挂载到同一个虚拟目录下；相当于分库分表这种，入口就是一个大表

![](/assets/image/container/docker/snipaste_20180820_231901.png)

上图是docker镜像的存储格式，一层一层的，就像集装箱

* bootfs 操作系统引导
* base image ： 具体的linux系统
* image ：软件，应用代码
* container ： 先忽略

除了 container 都是只读的，这种文件格式就叫镜像

## docker 容器
可以先把容器想象成一个虚拟机；整个就是一个文件系统。只有容器这一层是可写的；

可以保证，同一个镜像可以被多个容器运行

## docker仓库
有了镜像，把镜像传输到目的地。那么这个仓库就是用来存储镜像的，需要的地方从仓库拉取镜像

那么谁提供仓库呢？

* hub.docker.com   // docker官网提供，国内访问国外网速都差于是出现了第三方仓库
* c.163.com  // 网易云
* 自己搭建仓库  // 内部使用

## windows安装

windows10之外：
docker-toolbox ： 会虚拟一个docker运行环境

windows10 ：原生支持

下载地址官网查看即可；https://www.docker.com/products/docker-desktop

## macos
新版已经原生支持mac了；上面地址中查看；

macos中安装docker是所有平台中最方便的，直接拖进 application中即完成安装

## linxu

* redhat * centos
* ubuntu （都推荐在这个上面学习呢）

系统要求：64-bit os and version3.10

> redhat * centos 安装安靠
>
> http://www.imooc.com/article/16448
>
> [docker常用知识](chapter/container/docker/docker.md)

## 第一个docker镜像

从仓库拉取一个镜像，命令语法：
```bash
docker pull [OPTIONS] name:[TAG]

name: 镜像名称
TAG：镜像版本
```

查看本机有哪些镜像

```bash
docker images [OPTIONS] [REPOSITORY][:TAG]

REPOSITORY: 镜像名称；
后面的可选参数用得比较少，除非很多很多镜像的时候
```

拉取一个 hello-world 镜像；
```
docker pull hello-world

```

## 第一个容器
把下载的hello-world运行起来

运行命令：
```
docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARGS...]

COMMAND: 镜像运行起来的时候要执行什么命令
```

看到如下信息则标识运行成功了，下面有解释了为了打印这个消息，docker做了4步
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

```

运行过程
![](/assets/image/container/docker/snipaste_20180821_000949.png)

## 运行Nginx

实践前奏：

* 持久化运行的容器
* 前台挂起 & 后台运行
* 进入容器内部

```
网易云镜像中心去找，但是需要先登录才能访问：https://c.163yun.com/hub

docker pull hub.c.163.com/library/nginx:latest
```
先查看本机镜像
```
[root@localhost ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
docker.io/rabbitmq            3.7.7-management    df80af9ca0c9        2 weeks ago         149 MB
docker.io/hello-world         latest              2cb0d9787c4d        5 weeks ago         1.85 kB
hub.c.163.com/library/nginx   latest              46102226f2fd        16 months ago       109 MB

```
运行
```
docker run hub.c.163.com/library/nginx
```
敲完之后，终端阻塞了，这个应该是前台运行的，可以在另开一个终端使用 查看当前正在运行的容器
```
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED              STATUS              PORTS               NAMES
c3ccaf24ee0e        hub.c.163.com/library/nginx   "nginx -g 'daemon ..."   About a minute ago   Up About a minute   80/tcp              vibrant_almeida
```

后台运行直接加 -d参数即可,可以使用 `docker run --help`命令查看,其他命令也是一样，也可以使用help

```
docker run -d hub.c.163.com/library/nginx
```
进入容器，
```
# -it 分配一个伪终端。 29 是容器id部分字符串， bash 是要运行的命令
[root@localhost ~]# docker exec -it 29 bash
root@29dc6a24c527:/# which nginx
/usr/sbin/nginx

# 使用exit可以退出容器；
```

进入容器的感觉像是进入了一个linux服务器。但是我这里测试，发现好多命令永不了，不知道是怎么回事，包括  ll 显示目录的也不能使用

## docker 网络

* 网络类型 ：网络也是一种隔离类型
  - bridge ： 桥接模式（默认）
  - host ：与宿主机使用同一个
  - none
* 端口映射 ：容器内的网络是隔离的，所以需要一种手段来访问到容器内的网络端口

  ![](/assets/image/container/docker/snipaste_20180821_230931.png)
  大概意思就是：宿主机上的端口映射到容器内的端口

使用端口映射来运行nginx
```bash
# -p 端口映射：8080 是宿主机上的端口，80 是容器内的端口
# 运行之后，就可以通过访问宿主机8080端口进行访问容器内的nginx了
# -P 大写P,是把所有端口使用随机端口映射，可以通过docker ps 看到端口映射的详情
docker run -d -p 8080:80 hub.c.163.com/library/nginx

# 查看 8080端口是否ok
netstat -na|grep 8080
```

## 制作自己的镜像

* Dockerfile  ： 如何制作镜像，写代码
* docker build ： 构建
* Jpress：

  http://jpress.io/  下载这个博客应用，然后用这个项目来练手
  https://gitee.com/fuhai/jpress/tree/alpha/wars 下载这个

Dockerfile
```
# 继承一个镜像，这里找一个tomcat，意思是以tomcat为一个基础点
# docker pull hub.c.163.com/library/tomcat:latest
# 先让那个去下载，这里接着写，到时候就不用再下载了
from hub.c.163.com/library/tomcat:latest

# 容器信息
MAINTAINER liuguoguo xxx@163.com

# 把这个war文件copy到某一个地方
# 这里把之前下载的war复制到 tomcat中的一个目录中去
# 可以查看镜像仓库 刚才下载的地方有说明怎么使用的。里面有一个目录就是用来放war包的
COPY jpress-web-newest.war /usr/local/tomcat/webapps
```
这里我新建了一个目录，把Dockerfile和要打包的war都放在这个目录里面了，执行命令也是在这个目录里面

运行命令：`docker build .`

```
[root@localhost dockerdemo]# docker build .
Sending build context to Docker daemon  20.8 MB
Step 1/3 : FROM hub.c.163.com/library/tomcat:latest
 ---> 72d2be374029
Step 2/3 : MAINTAINER liuguoguo xxx@163.com
 ---> Using cache
 ---> af2a0019bc2d
Step 3/3 : COPY jpress-web-newest.war /usr/local/tomcat/webapps
 ---> 75a573e4e281
Removing intermediate container 9c88b6be1ef3
Successfully built 75a573e4e281
```
可以看到执行了3个步骤；最后一步是把镜像移动到了 docker中；

查看镜像，发现出现一个 none的镜像；
```
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
<none>                         <none>              75a573e4e281        54 seconds ago      313 MB
```

这个是打包的时候没有指定名称和版本；重新编译下
```
docker build . -t jpress:latest

# 可以使用--help命令查看docker build 的帮助信息
```
再次查看
```bash
[root@localhost dockerdemo]# docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
jpress                         latest              75a573e4e281        3 minutes ago       313 MB

```

## 运行自己的容器

镜像打好了，来运行这个镜像

```bash
# 镜像内部端口需要查看tomcat默认开放的端口是什么，因为我们之前没有做任何的修改
docker run -d -p 8080:8080 jpress

# 查看是否已经运行
docker ps

# 查看8080端口是否已经使用
netstat -na|grep 8080
```

访问地址：http://localhost:8080/ 可以看到tomcat的页面了。

由于默认的tomcatwar 是增加了项目名；所以访问：http://localhost:8080/jpress-web-newest (war包名就是这个，所以项目名就是war包名)

已经打开了，使用该博客打开就有一个引导，需要填写数据库；

这里在docker中运行一个mysql
```bash
# -e 是设置环境参数，这个是在镜像描述使用说明里面看到的
# 这样可以设置一个root的用户名和密码
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=jpress hub.c.163.com/library/mysql:latest
```

结果也没有在该博客配置mysql的地方连接上mysql。在windows中连接mysql信息能连接上；

进入jpress容器ping宿主机也能ping通,但是jpress的日志一直报错连接不上

## 总结

* 集装箱，标准化，隔离
* 镜像、容器、仓库（BUILD SHIP RUN）
* docker命令 pull,build,run,stop,restart,exec...
