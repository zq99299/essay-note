# 开启APR

> 摘抄至：http://blog.csdn.net/clementad/article/details/47320037

APR：Apache Portable Run-time libraries，Apache可移植运行库
在早期的Apache版本中，应用程序本身必须能够处理各种具体操作系统平台的细节，并针对不同的平台调用不同的处理函数。随着Apache的进一步开发，Apache组织决定将这些通用的函数独立出来并发展成为一个新的项目。这样，APR的开发就从Apache中独立出来，Apache仅仅是使用APR而已。

Tomcat Native：这个项目可以让 Tomcat 使用 Apache 的 apr 包来处理包括文件和网络IO操作，以提升性能。

## Linux下，Tomcat启用APR需要三个组件：
* apr
* apr-util
* tomcat-native.tar.gz（Tomcat自带，在bin目录下）


1、查看是否已经安装了apr和apr-util

```bash
# rpm -qa apr
apr-1.4.8-3.el7.x86_64

# rpm -qa apr-util
apr-util-1.5.2-6.el7.x86_64
```

2、查看是否有最新版的apr和apr-util

```bash
# yum list | grep apr
apr.x86_64                              1.4.8-3.el7                    @anaconda
apr-util.x86_64                         1.5.2-6.el7                    @anaconda
```

3、如果还没安装，用yum安装：

```bash
# yum install apr-devel apr apr-util
```

4、安装tomcat-native：
搜索tomcat-native安装包：

```bash
# yum list | grep tomcat-native


如果已经存在，直接安装：
# yum install tomcat-native
……
  正在安装    : tomcat-native-1.1.30-1.el7.x86_64        1/1
  验证中      : tomcat-native-1.1.30-1.el7.x86_64         1/1

已安装:
  tomcat-native.x86_64 0:1.1.30-1.el7                                                                                                                                                        
完毕！
```

查看是否安装成功：

```bash
# rpm -qa tomcat-native
tomcat-native-1.1.30-1.el7.x86_64
````

配置相关的全局变量：就是普通的把/usr/local/apr/lib路径添加到环境变量中

```bash
# vi /etc/profile
添加：export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib
# source /etc/profile
```

5、重启Tomcat，看看是否可以成功使用APR
如果一切正常：
APR启动：

```bash
[main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-apr-18080"]
[main] org.apache.catalina.startup.Catalina.start Server startup in 13617 ms
相比NIO模式的启动，速度快了一些（~15%）：
NIO启动：
[main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-18080"]
[main] org.apache.catalina.startup.Catalina.start Server startup in 15671 ms
```

如果发现异常log，比如：

```bash
06-Aug-2015 14:46:04.949 SEVERE [main] org.apache.catalina.core.AprLifecycleListener.init An incompatible version 1.1.30 of the APR based Apache Tomcat Native library is installed, while Tomcat requires version 1.1.32
```

说明系统自带的tomcat-native版本太低。
删除：

```bash
# yum erase tomcat-native
```

用yum检查有没有最新版：

```bash
# yum update tomcat-native
如果yum找不到最新版，则下载或从Tomcat/bin中解压安装。

从Tomcat/bin目录中，解压tomcat-native.tar.gz文件：
# tar -zxvf tomcat-native.tar.gz
得到文件夹：tomcat-native-1.1.33-src
# cd tomcat-native-1.1.33-src/jni/native/
# ./configure --with-apr=/usr/local/apr    （官网中例子的其他参数不需要，会自动找到）
# make && make install
```

参考：
官网的安装指导：http://tomcat.apache.org/native-doc/
Tomcat Connector三种运行模式（BIO, NIO, APR）的比较和优化：http://blog.csdn.net/clementad/article/details/47045673