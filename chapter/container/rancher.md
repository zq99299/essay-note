# rancher
> https://rancher.com
>
> https://www.cnrancher.com/
>
> https://www.cnrancher.com/quick-start/

作用：更方便的管理docker

## quick-start
quick-start 中只需要两部。

1. 在一台linux系统上装docker，并运行docker
2. 运行命令`$ sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher`
3. 访问这台服务器 `https://<server_ip>`


进入主页之后：

1. 提示绑定服务器地址
2. 重置admin的密码，默认就是admin
3. 进入主页后，右下角可以切换语言为中文
4. 添加一个集群。名称自定义。选择 添加主机自建Kubernetes集群（也就是那个custom的图标）
5. 添加集群主机（如果不小心关闭了最开始的提示界面，可以点击某个集群编辑，然后就能看到如下图所示）

  ![](/assets/image/container/snipaste_20180820_221357.png)
  需要在另外的机器上执行这个命令，server 和 worker 不要放在一台上面


执行这个命令后，注册的主机没有任何信息；通过 docker log 容器id 查看到的日志显示:rancher waiting for node to register

下面两个地址也是出现了这样的情况，不知道该怎么继续下去了

```
https://forums.rancher.com/t/worker-node-waiting-to-register-in-vagrant-virtualbox/10078

https://github.com/rancher/rancher/issues/11365
```

最后加了qq群。有人指点：
```
1.关闭seliunx
2.永久修改hostName
3.固定内网IP，并关闭防火墙（官网有具体放行的端口，测试可以直接关闭防火墙）
4.修改时区和时间
5.重启服务器以上4点挨个验证
```

* 关闭seliunx
  ```bash
  # 查看 seliunx 状态
  getenforce

  # 修改配置文件关闭
  vim /etc/selinux/config

  # This file controls the state of SELinux on the system.
  # SELINUX= can take one of these three values:
  #     enforcing - SELinux security policy is enforced.
  #     permissive - SELinux prints warnings instead of enforcing.
  #     disabled - No SELinux policy is loaded.
  SELINUX=enforcing    把这里修改成 disabled
  # SELINUXTYPE= can take one of three two values:
  #     targeted - Targeted processes are protected,
  #     minimum - Modification of targeted policy. Only selected processes are protected.
  #     mls - Multi Level Security protection.
  SELINUXTYPE=targeted

  ```
* 永久修改hostName

  ```bash
  # 打开后修改成唯一命名，要参与集群的机器都不要重名
  vim /etc/hostname
  ```
* 固定内网ip

  这个在ui里面修改的很方便。在上面hostName的时候，就可以使用固定字符串+ip后缀的方式来命名
* 修改时区和时间

  CentOS7、RHEL7、Scientific Linux 7、Oracle Linux 7
  ```bash
  最好的方法是使用timedatectl命令

  timedatectl list-timezones |grep Shanghai    #查找中国时区的完整名称
  # Asia/Shanghai
  timedatectl set-timezone Asia/Shanghai    #其他时区以此类推
  ```

如果想重新下载或则有问题的时候，可以执行清理脚本，清理掉所有的容器

cleanup.sh
```bash
#!/bin/sh
docker rm -f $(docker ps -qa)
docker volume rm $(docker volume ls -q)
cleanupdirs="/var/lib/etcd /etc/kubernetes /etc/cni /opt/cni /var/lib/cni /var/run/calico /opt/rke"
for dir in $cleanupdirs; do
  echo "Removing $dir"
  rm -rf $dir
done
```

注意：

在复制主机命令的时候：

第一台主机：勾选角色至少是 etcd  Control ，  Worker可选。这个是在server的日志中看到的，等待他们两个角色注册；
第二台主机：可以只选worker角色添加；

如下图：
  * dc03运行了3个角色，在顶部显示了一条红色信息

    通过 docker logs 查看运行的3个角色的日志信息，能看到一条报错信息 no souch container:kubelet
    目前还不知道是什么意思
  * dc04就只运行了一个wokere的角色

    查看日志就一直显示rancher waiting for node to register
![](/assets/image/container/snipaste_20180821_225033.png)

后来看了下官网的基础文档，可能是需要自己搭建k8s集群，然后使用rancher来管理；

开启三个角色的机器上，过一会，会在webui中显示如下图一行的红色提醒；这里提醒一个下载一个，看看最后能不能让它跑起来

![](/assets/image/container/snipaste_20180822_010853.png)

```
docker pull rancher/rke-tools:v0.1.13
docker pull rancher/hyperkube:v1.11.1-rancher1
```
下载完成这两个，发现ui中的红色信息变化了，最后部署ok了 停留在下面的图中
![](/assets/image/container/snipaste_20180822_011327.png)
