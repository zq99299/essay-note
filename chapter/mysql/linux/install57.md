# Mysql-linux在线安装5.7

## CentOS 通过yum在线安装MySQL5.7

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

## 更改mysql存储位置
首先要关闭mysql

```bash
service mysqld stop
```

转移数据

```bash
cp -a /var/lib/mysql /home/mysql_data/
```

接下来修改脚本和配置文件中用到的三个文件中的路径,主要目的是修改 datadir 的路径为 上面转移数据的目标路径

```bash
vi /etc/my.cnf        # 修改两处- datadir和socket的前缀路径
vi /etc/init.d/mysqld    # 修改多处，需要用搜索查找/var/lib/mysql的路径，并替换掉
vi /usr/bin/mysqld_safe  # 修改一处，datadir
```

下面需要建立一个mysql.sock的链接：

```bash
#ln -s /data/mysql_data/mysql/mysql.sock /var/lib/mysql/mysql.sock
```

修改完成，启动mysql 服务

```bash
service mysqld start
```

如果是新安装的mysql的话，启动的时候会出现以下提示,这个不用担心会自动生成的。
```bash
/sbin/restorecon:  lstat(/data/mysql_data/mysql) failed:  No such file or directory
Initializing MySQL database:                               [  OK  ]
Starting mysqld:                                           [  OK  ]
```

## 5.7 初始密码修改
```bash
vim /etc/my.cnf

加上这一行，关闭登录验证
skip-grant-tables=1  
```

重启mysql,然后登录到mysql，进行修改密码
```bash
# service mysql restart
# mysql

// mysql 链接上之后，切换到mysql库

mysql > use mysql
mysql > UPDATE user SET authentication_string=PASSWORD("123456") WHERE user='root';   
mysql > update user set password=password('123456') where user='root'; 
```
第二条修改如果报错的话，没有关系。

然后把登录验证打开。重启mysql即可
