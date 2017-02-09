# redis集群的准备
## zlib
1. 安装redis-cluster依赖:redis-cluster的依赖库在使用时有兼容问题,在reshard时会遇到各种错误,请按指定版本安装.
2. 确保系统安装zlib,否则gem install会报(no such file to load -- zlib)
 
```bash
wget http://heanet.dl.sourceforge.net/project/libpng/zlib/1.2.6/zlib-1.2.6.tar.gz
tar xzf ruby-1.9.2-p290.tar.gz 
./configure  
make  
make install  
```
## 安装ruby
yum install ruby

## 安装rubygems
yum install rubygems


## 安装redis cluster 和 配置config  
1. gem install redis
2. 下载redis指定版本，集群模式（redis3.0 以上版本）
```bash
rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/jemalloc-3.6.0-1.el6.x86_64.rpm
rpm -ivh ftp://fr2.rpmfind.net/linux/remi/enterprise/6/remi/x86_64/redis-3.2.3-1.el6.remi.x86_64.rpm

wget http://download.redis.io/releases/redis-3.2.3.tar.gz 
cp redis-3.2.3/src/redis-trib.rb /usr/local/bin/redis-trib 
```
共三台机器；每台机器装两个redis实例

### 准备config文件
**每个实例的结构如下：**

```bash
mkdir -p /data/redis/cluster
	-|6379
	  -|log    			
	  -|working
	  -|redis.config			
	-|7000
	  -|log
	  -|working	
	  -|redis.config		

```

#### 4.1.1 首次搭建集群修改的配置项（创建集群之前）

|  配置项  		  | 描述			
|-----------------|---------------------------------------------
| bind 			  | 绑定本机ip，创建集群的时候会用到此ip，通过此ip进行通信
| port 			  | 绑定实例端口，注：防火墙必须放行此端口
| cluster-enabled | 开启集群模式
| pidfile 		  | pid文件路径

**redis.config**
```bash
bind 192.168.7.225
port 7000
cluster-enabled yes
pidfile /var/run/redis_7000.pid  #按端口进行配置
```

**以下是redis3.2.1的默认配置项（具体redis.config里面是不是必须添加这些默认项待测）**
```bash

logfile ""
dir ./

protected-mode yes
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
loglevel notice
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
cluster-config-file nodes.conf
cluster-node-timeout 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes

```


### 4.2 按config启动每个实例
```bash
cd /data/redis/cluster/6379/
redis-server redis.conf 
cd /data/redis/cluster/7000/
redis-server redis.conf 
```
`注：`redis-server 直接进入redis安装目录启动两个实例的话，偶尔会报错，暂时不知道原因，所以配置成全局的命令


### 4.3 创建集群
初次需要创建集群，使用以下命令，前提是：
1. 防火墙关闭（会有一些通信端口）
2. ruby和rubygems命令能正常使用：（如：gem install redis能安装的话，一般都行）

```bash
./redis-trib.rb  create --replicas 1 192.168.7.225:6379 192.168.7.63:6379 192.168.7.225:7000 192.168.7.63:7000 192.168.7.213:6379 192.168.7.213:7000
```

`注意：` 4.1.1 中的配置项在此步骤之前建议严格遵守，因为如果在创建集群之前配置了密码，那么你创建集群就会报错：链接不上某台节点

## 进阶配置config（创建完集群之后）
|  配置项  		  				| 描述			
|-------------------------------|---------------------------------------------
| requirepass 	  				| 设置redis实例的访问密码
| masterauth 	  				| 主从通信密码，so:所有集群要么设置一样的密码，要么都不设置密码
| logfile						| 日志文件路径
| dir							| 工作目录路径，aof文件存储什么的
| daemonize						| 后台执行，不阻塞命令行

```bash
requirepass qwe123
masterauth qwe123
logfile ./log/redis.log				#当前目录是按照redis.config为参照物，so:这里可以使用./ 来简化配置
dir ./working
daemonize yes
```

`小技巧：` 不设置后台后台运行，和日志路径，直接ctrl+c就能kill掉当前redis实例，方便调试配置项

## 错误解决
### 防火墙,暴露端口（应该还有其他的端口需要。）
```bash
vim /etc/sysconfig/iptables
-A INPUT -p tcp -m tcp --dport 7000 -j ACCEPT

service iptables stop
service iptables start
```

### 解决启动警告(临时解决)
echo 1 > /proc/sys/vm/overcommit_memory
echo 511 > /proc/sys/net/core/somaxconn
echo never > /sys/kernel/mm/transparent_hugepage/enabled




# redis集群的管理
redis-trib.rb 命令，用来管理集群，在redis安装目录下的 src中
参考文档：http://blog.csdn.net/huwei2003/article/details/50973967

## trib 配置
前面说道，创建集群之前不要设置密码，原因是 trib的命令是不带密码的，所以不能设置，这里可以修改配置来支持密码：
配置文件所在的路径（默认在线安装）：/usr/lib/ruby/gems/1.8/gems/redis-3.2.1/lib/redis/client.rb

```bash
require "redis/errors"
require "socket"
require "cgi"

class Redis
  class Client

    DEFAULTS = {
      :url => lambda { ENV["REDIS_URL"] },
      :scheme => "redis",
      :host => "127.0.0.1",
      :port => 6379,
      :path => nil,
      :timeout => 5.0, 
      :password => nil,    #密码修改成集群密码
      :db => 0,
      :driver => nil,
      :id => nil,
      :tcp_keepalive => 0,
      :reconnect_attempts => 1,
      :inherit_socket => false
    }

``` 

## 关闭单个服务
```bash
redis-cli [-c] -h 192.168.1.185 -p 7000 shutdown 
```

## redis-cli
```bash
-c 进入集群模式:如获取213机器上的一个key，如果该机器上不存在，则会自动跳转到其他有的机器上面
redis-cli -c -h 192.168.7.213
```

## 创建集群create
初次需要创建集群，使用以下命令，前提是：
1. 防火墙关闭（会有一些通信端口）
2. ruby和rubygems命令能正常使用：（在准备步骤中4.1中使用gem install redis能安装的话，一般都行）

```bash
./redis-trib.rb  create --replicas 1 192.168.7.225:6379 192.168.7.63:6379 192.168.7.225:7000 192.168.7.63:7000 192.168.7.213:6379 192.168.7.213:7000
```

## 检查集群状态check
 ```bash
#redis-trib.rb的check子命令构建  ,在redis安装目录下的 src下
#ip:port可以是集群的任意节点  
./redis-trib.rb check 192.168.1.185:6380 
```

## 查看集群信息info

info命令用来查看集群的信息。info命令也是先执行load_cluster_info_from_node获取完整的集群信息。然后显示ClusterNode的info_string结果，示例如下：
```bash
[root@i-vgjivpza src]#  ./redis-trib.rb info 192.168.7.213:6379
192.168.7.213:6379 (4b828f29...) -> 0 keys | 5461 slots | 1 slaves.
192.168.7.225:6379 (2952d28b...) -> 0 keys | 5461 slots | 1 slaves.
192.168.7.63:6379 (b8731b08...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
```

##add-node将新节点加入集群
add-node命令可以将新节点加入集群，节点可以为master，也可以为某个master节点的slave。
```bash
add-node    new_host:new_port existing_host:existing_port
          --slave
          --master-id <arg>
```
**add-node有两个可选参数：**
--slave：设置该参数，则新节点以slave的角色加入集群
--master-id：这个参数需要设置了--slave才能生效，--master-id用来指定新节点的master节点。如果不设置该参数，则会随机为节点选择
master节点。

```bash
把213的新节点添加到63这个集群中去
./redis-trib.rb add-node --slave 192.168.7.213:7001 192.168.7.63:6379
>>> Adding node 192.168.7.213:7001 to cluster 192.168.7.63:6379
>>> Performing Cluster Check (using node 192.168.7.63:6379)
M: 1a05ee8d716c5f8b5def33c28f23fa398004fb30 192.168.7.63:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: c215761aef6ab6f6c6bc473bc0f5c4bd4136d716 192.168.7.213:7000
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: e34e3d1f221c2d9afc842df69837429ff5e62990 192.168.7.225:6379
   slots: (0 slots) slave
   replicates c215761aef6ab6f6c6bc473bc0f5c4bd4136d716
S: 5a36b6ed0f83626ab23d67f3fe427cdf3bfacc69 192.168.7.213:6379
   slots: (0 slots) slave
   replicates eed38f479321100ece02b33fe68afb7181d2f219
S: 2edf0911caf0b99b0589fca1cf989ed1c1459e25 192.168.7.63:7000
   slots: (0 slots) slave
   replicates 1a05ee8d716c5f8b5def33c28f23fa398004fb30
M: eed38f479321100ece02b33fe68afb7181d2f219 192.168.7.225:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
Automatically selected master 192.168.7.63:6379
>>> Send CLUSTER MEET to node 192.168.7.213:7001 to make it join the cluster.
Waiting for the cluster to join.
>>> Configure node as replica of 192.168.7.63:6379.
[OK] New node added correctly.

```
成功之后，可以使用check命令查询到集群状态中多了一个新的slave节点









