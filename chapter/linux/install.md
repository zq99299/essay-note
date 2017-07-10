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
node 8+ 的包后缀是xz结尾的。所以要先安装xz
```bash
yum install xz
# 解压
xz -d nodexxx

# 解压完成之后，发现。后缀没有了。再使用
# 配置环境变量
export NODE_HOME=/mnt/sit/app/node-v8.1.3-linux-x64
export PATH=$NODE_HOME/bin:$PATH
```

## nginx