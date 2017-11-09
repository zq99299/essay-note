# 监控相关

## sigar 的使用

最主要的是得到那几个dll文件，下载地址：https://sourceforge.net/projects/sigar/files/latest/download?source=files

官网下载 hyperic-sigar-1.6.4.zip

解压之后在 hyperic-sigar-1.6.4\sigar-bin\lib 该目录中找到以下windows和linux相关需要的文件
```
libsigar-amd64-linux.so
libsigar-ia64-linux.so
libsigar-ppc64-linux.so
libsigar-ppc-linux.so
libsigar-s390x-linux.so
libsigar-x86-linux.so
sigar-amd64-winnt.dll
sigar-x86-winnt.dll
```

想让程序找到这个依赖文件有以下几种办法：
1. 然后打包的时候打包libs包中，web项目在web-info/libs中，jar项目在classpath下（这种方法比较费劲）
2. 拷贝到jdk安装目录的bin下，比如：`jdk1.8.0_45\bin` （这种方法是最一劳永逸的）