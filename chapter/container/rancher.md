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

  ![](/assets/image/imooc/spring_cloud/snipaste_20180820_221357.png)
  需要在另外的机器上执行这个命令，server 和 worker 不要放在一台上面
