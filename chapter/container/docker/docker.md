# docker

## docker 安装
> 参考官网文档 ： https://docs.docker.com/install/linux/docker-ce/centos/

使用yum包安装

使用有sudo权限的帐号登录系统。


1. 更新yum包。
  ```
  $ sudo yum update
  ```
2. 设置存储库

  ```
  $ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2

  $ sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
  ```

3. 安装ce版docker

  ```
  $ sudo yum install docker-ce
  ```
4. 启动docker

  ```
  $ sudo systemctl start docker
  ```
5. 运行hellow-world镜像查看docker是否正确安装

  ```
  $ sudo docker run hello-world
  ```

## 常用命令
1. 停止停止所有的container

  ```
  docker stop $(docker ps -a -q)
  ```
  也可以协商imagesid 来停止指定的容器
2. 查看镜像列表
  ```
  docker images
  ```
3. 删除镜像

  ```
  docker rmi 147051a21fd9
  ```

  如果提示：（must be forced）类似的提示，不能删除的话，可以先执行以下命令，再执行删除操作
  ```
  docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker stop

  docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker rm
  ```

## 配置 docker镜像加速器

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
