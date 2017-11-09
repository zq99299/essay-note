# windows下Redis的使用


## 安装
`redis`官网没有windows版本，由微软开源团队维护。https://github.com/MSOpenTech/redis/releases

下载 `.msi` 文件之后安装

## 启动
进入到安装目录下执行
```bash
redis-server  redis.windows.conf  
```
如果报错: creating server tcp listening socket 127.0.0.1:6379: bind No error
执行以下操作;
```bash
1. Redis-cli.exe
2. shutdown
3. exit
4. redis-server.exe redis.windows.conf


参考连接:http://blog.csdn.NET/fengzhihen2007/article/details/52211048
```