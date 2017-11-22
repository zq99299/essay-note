#CentOS 通过yum在线安装MySQL5.7

Step1: 检测系统是否自带安装mysql

```bash
# yum list installed | grep mysql
```

Step2: 删除系统自带的mysql及其依赖
命令：
```bash
# yum -y remove mysql-libs.x86_64
```

Step3: 给CentOS添加rpm源，并且选择较新的源
命令：
```bash
# wget dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
# yum localinstall mysql-community-release-el6-5.noarch.rpm
# yum repolist all | grep mysql
# yum-config-manager --disable mysql55-community
# yum-config-manager --disable mysql56-community
# yum-config-manager --enable mysql57-community-dmr
# yum repolist enabled | grep mysql
```

Step4:安装mysql 服务器
命令：

```bash
# yum install mysql-community-server
```

Step5: 启动mysql
命令:
```bash
# service mysqld start
```

Step6: 查看mysql是否自启动,并且设置开启自启动
命令:

```bash
# chkconfig --list | grep mysqld
# chkconfig mysqld on
```

Step7: mysql安全设置
命令：

```bash
# mysql_secure_installation
```