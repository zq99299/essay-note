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

就是最后初始化的时候，除了openssl需要指定地址外。剩下的不用再指定了。会自动找到的
装成功之后的目录是在 初始化显示的 `nginx path prefix: "/usr/local/nginx"` 目录中


```bash
# 启动
sudo /usr/local/nginx/nginx
# 如果报错  /usr/local/nginx/nginx: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory

## 先查看
ldd $(which /usr/local/nginx/sbin/nginx)

[root@localhost lib]# ldd $(which /usr/local/nginx/sbin/nginx)
	linux-vdso.so.1 =>  (0x00007ffe91ffd000)
	libdl.so.2 => /lib64/libdl.so.2 (0x0000003123e00000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x0000003124600000)
	libcrypt.so.1 => /lib64/libcrypt.so.1 (0x0000003128a00000)
	libpcre.so.1 => not found
	libz.so.1 => /lib64/libz.so.1 (0x0000003124a00000)
	libc.so.6 => /lib64/libc.so.6 (0x0000003124200000)
	/lib64/ld-linux-x86-64.so.2 (0x0000003123a00000)
	libfreebl3.so => /lib64/libfreebl3.so (0x0000003129200000)

# 可以看到上面的确是未找到。
# 手动关联一下
cd /lib
ln -s /lib/libpcre.so.0.0.1 /lib/libpcre.so.1
# 会发现还是没有找到.有可能你的机器是x64的。执行以下代码
ln -s /usr/local/lib/libpcre.so.1 /lib64/
```

## 启动命令等
指定配置文件启动：
```bash
cd /usr/local/nginx
./nginx -c ./conf/nginx.conf
```
修改了配置文件，重启
```bash
./nginx -s reload
```
停止服务
```
./nginx -s stop
```

## nginx vue History模式配置
```bash

#user  nobody;
worker_processes  auto;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  2048;
    multi_accept on;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;
    gzip_disable "msie6";
    # gzip_static on; 
    gzip_proxied any;
    gzip_min_length 1000; 
    gzip_comp_level 4; 
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /mnt/prod/release/vue/shopmarketing-vue;
            # index  index.html index.htm;
			# history模式下使用，不然会被转到404
            try_files $uri $uri/ /index.html =404;
        }
		# 拦截api开头的请求，代理到目标地址
		location ^~ /api/ {
            proxy_pass http://127.0.0.1:18801;
			client_max_body_size 500M;
        }
		# 如果nginx和文件存储服务器在一起，就可以直接拦截映射到目标文件
        location ^~ /download/ {
            root  /;
			add_header Content-Disposition: 'attachment;';
            rewrite ^/(download)/(.*)$ /mnt/sit/release/resources/shopmarketing/$2 break;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

```
