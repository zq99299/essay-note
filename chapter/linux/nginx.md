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
首先进入nginx安装目录，下面的命令都是在该目录中执行的
```bash
cd /usr/local/nginx
# 如果执行不了nginx命令的话就进入sbin目录
cd /usr/local/nginx/sbin
```
测试配置文件是否有异常
```bash
./nginx -t
``
指定配置文件启动：
```bash
# 这里的conf文件使用的相对路径需要区分当前目录是sbin还是根目录
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

## 通过 yum 安装

```
# 添加官网的源地址
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# 6 版本装以下的 通过 lsb_release -a 查询版本
sudo rpm -Uvh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
# 安装依赖
sudo yum -y install gcc automake autoconf libtool make
sudo yum install -y nginx
```

以下是Nginx的默认路径：

(1) Nginx配置路径：/etc/nginx/

(2) PID目录：/var/run/nginx.pid

(3) 错误日志：/var/log/nginx/error.log

(4) 访问日志：/var/log/nginx/access.log

(5) 默认站点目录：/usr/share/nginx/html

nginx   启动

nginx -t  测试命令

nginx -s reload 修改nginx.conf之后，可以重载

## 获得免费的 ssl 证书 https
```bash
server {
        listen 80;
        server_name mrcode.com;
        charset utf-8;

        #access_log  logs/host.access.log  main;
        #gzip 参考 http://web.facesoho.com/server/nginx-precompression.html
        #启动预压缩功能，对所有类型的文件都有效，优先找到xx.gz 的文件
        gzip_static on;
        #找不到预压缩文件，进行动态压缩
        gzip on;
        gzip_min_length 3k;
        gzip_buffers 4 16k;
        #gzip_http_version 1.0;
        gzip_comp_level 2;
        gzip_types text/plain application/x-javascript application/javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        gzip_vary off;
        gzip_disable "MSIE [1-6]\.";

        location / {
                root   /mnt/service/tlz-shopify-plugin-edm-app;
                try_files $uri $uri/ /index.html =404;
                # 测试功能，发布新版本后，不需要强刷，只需要按 f5 刷新，就能获取新版本
                if ($request_filename ~* .*\.(?:htm|html)$){
                        add_header Cache-Control "private, no-store, no-cache, must-revalidate, proxy-revalidate";
                }
                if ($request_filename ~* .*\.(?:js|css)$){
                        expires      7d;
                }
                if ($request_filename ~* .*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$){
                        expires      7d;
                }
        }
        # 拦截api开头的请求，代理到目标地址
        location ^~ /api/ {
                        #设置主机头和客户端真实地址，以便服务器获取客户端真实 IP
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                #禁用缓存
                proxy_buffering off;
                proxy_pass http://localhost:15000;
                client_max_body_size 500M;
        }
        # 这里的信息是下面的 certbot-auto 自动添加的
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/em.pluginappstore.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/em.pluginappstore.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}        
```

certbot-auto 使用方法

```bash
# 前提是：你的域名 80 端口要先配置好，执行后会自动找到 80 端口的文件，然后帮你添加
wget https://dl.eff.org/certbot-auto
sudo mv certbot-auto /usr/local/bin/certbot-auto
## 修改为你当前的用户
sudo chown root /usr/local/bin/certbot-auto
sudo chmod 0755 /usr/local/bin/certbot-auto                    
# 在这期间，会要求你输入你的邮箱、域名
sudo certbot-auto --nginx   

certbot-auto renew 重新更新
```

crontab 脚本 3 个月更新一次证书

```bash
crontab -e
# 每 3 个月的月末  23:50 分 执行一次
# 通过调整 2、5、8、11 月
50 23 28-31 2,5,8,11 * [ `date -d tomorrow +\%e` -eq 1 ] && /updatehttps.sh > /updatehttps.logs
```
