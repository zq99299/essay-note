# 运行第一个docker容器

> 一个docker的入门课程：《第一个docker化的java应用》https://www.imooc.com/learn/824

> 网易云镜像中心 https://c.163yun.com/hub

对eureka这个项目进行镜像编译

在该项目根目录下创建文件：Dockerfile
```
# 从网易云上获取一个jdk镜像https://c.163yun.com/hub#/m/repository/version/?repoId=2999
# 右侧就是下面的链接，java:后面的是版本名称。
FROM hub.c.163.com/library/java:8-alpine

# 添加项目下的jar，使用gradle task bootJar 打包的
ADD build/libs/*.jar app.jar

# 暴露的端口
EXPOSE 8761

# 执行的命令：运行这个jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

编译镜像
```
docker build -t springcloud2
```
启动
```
docker run -p 8761:8761 -d springcloud2/eureka
```

到这里还是一脸懵逼啊，看来要先去学习下 docker的基本上使用才行


## rancher
该章节请参考：[容器相关知识](/chapter/container/index.md)

该章节老师并没有详细解说，是直接在他已经弄好的机器上演示的。
并不是他演示那样（也就是下面笔记记录的这个），就能看到效果的

> https://rancher.com
>
> https://www.cnrancher.com/
>
> https://www.cnrancher.com/quick-start/

作用：更方便的管理docker

### quick-start
quick-start 中只需要两部。

1. 在一台linux系统上装docker
2. 运行命令`$ sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher`
3. 访问这台服务器 `https://<server_ip>`


### 配置 docker镜像加速器

> https://www.cnblogs.com/zhxshseu/p/5970a5a763c8fe2b01cd2eb63a8622b2.html

使用了阿里云的镜像仓库： https://dev.aliyun.com/search.html

```
1. 安装／升级Docker客户端

推荐安装1.10.0以上版本的Docker客户端，参考文档 docker-ce

2. 配置镜像加速器

针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://you加速地址.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
