# nginx 

## 安装

### 官网下载好压缩包，并解压

### 安装依赖
```bash
yum -y install gcc automake autoconf libtool make
```

### 执行 ./configure 命令进行初始化
进入安装目录运行  ./configure  进行初始化
```bash
./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```
如果出现错误：首先尝试nginx目录下的自带依赖包
```bash
# 查找pcre的路径。找到之后追加到初始化命令后面
$ find / -name pcre  

# 先尝试下面的命令是否能成功，如果不能成再追加 --with-zlib=/mnt/sit/app/nginx-1.13.2/auto/lib/zlib
$ ./configure --with-pcre=/mnt/sit/app/nginx-1.13.2/auto/lib/pcre --with-zlib=/mnt/sit/app/nginx-1.13.2/auto/lib/zlib --with-openssl=/mnt/sit/app/nginx-1.13.2/auto/lib/openssl


```

如果安装成功则会显示如下信息
```bash
...
Configuration summary
  + using PCRE library: /mnt/sit/app/nginx-1.13.2/auto/lib/pcre
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

```
如果安装失败。请手动安装倚赖包
```bash
1. 安装pcre
    1. 检查CentOS系统是否安装prce，，如果已安装则会显示pcre的版本信息
        [root@localhost /]# rpm -qa pcre
        pcre-7.8-6.el6.i686
    2.删除pcre包
        [root@localhost /]# rpm -e --nodeps pcre
        [root@localhost /]# rpm -qa pcre
    3.在线安装pcre
        [root@localhost /]# yum install pcre
    4.查看pcre的安装路径
        [root@localhost /]# rpm -qa pcre
    pcre-7.8-6.el6.i686
        [root@localhost /]# rpm -ql pcre-7.8-6.el6.i686
2. 安装编译依赖库yum install gcc gcc-c++ openssl openssl-devel zib-devel zib 	
```

安装成功之后，执行命令
```bash
$ make install
```

## 上面的安装依然失败了，直接按照官网的安装文档
http://www.nginx.cn/install

亲测装成功了。

装成功之后的目录是在 初始化显示的 `nginx path prefix: "/usr/local/nginx"` 目录中