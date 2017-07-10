# 常用软件安装

系统环境变量文件：
```bash
/etc/profile
```
刷新配置文件
```bash
source /etc/profile
```

## jdk
环境变量配置：

```bash
export JAVA_HOME=/mnt/sit/app/jdk1.8.0_111/
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

## gradle
环境变量配置：
```bash
export GRADLE_HOME=/mnt/sit/app/gradle-3.2.1
# 这个是指定仓库文件下载到哪里,没有特殊需求可以不配置
export GRADLE_USER_HOME=/home
export PATH=$GRADLE_HOME/bin:$PATH

```

## node
node 包后缀是xz结尾的。所以要先安装xz
```bash
yum install xz
# 解压
xz -d nodexxx

# 解压完成之后，发现。后缀没有了。再使用
# 配置环境变量
export NODE_HOME=/mnt/sit/app/node-v8.1.3-linux-x64
export PATH=$NODE_HOME/bin:$PATH
```
### 修改缓存和修改淘宝源
命令行的方式修改：
```bash
npm config set prefix “D:\Program Files\node\node-global”
```

直接修改文件.npmrc.该文件地址在 /当前按照用户 目录下。如果执行了上面的命令就会生成一个。没有的话自己创建
使用 `npm -h` 命令也会告诉你.npmrc的文件地址在哪里
```
prefix=/mnt/sit/app/node/node-global
cache=/mnt/sit/app/node/node-cache
registry = https://registry.npm.taobao.org  # 修改成淘宝的镜像源，加快国内的构建速度
```


## nginx

## git安装
```bash
yum install git 
```
