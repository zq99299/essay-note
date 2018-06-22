# 开启Jconsloe

开启TOMCAT的支持JMX性能分析；在windows链接远程TOMCAT的配置

编辑文件：catalina.sh ；在最开始追加以下内容
```
CATALINA_OPT="-Djava.rmi.server.hostname=xx.11.11.11 -Dcom.sun.management.jmxremote.port=9004 -Dcom.sun.management.jmxremote.rmi.port=9004 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
```

注意事项：在TOMCAT7下面发现CATALTA_OPT无效

* CATALTA_OPT 需要换成 JAVA_OPTS； 也就是说需要把命令增加到JAVA_OPTS后面（对于这个不生效不知道是不是刚开始没有加hostname的原因）
* hostname 必须增加；写上你的外网ip；

该配置无用户名密码，用户名密码的链接还需要去百度

## 其他
也可以在 setenv.sh文件中写。
因为catalina.sh里面有这段代码，这是tomcat为了让用户自定义jvm参数而不破坏源文件的办法。

```bash
# Ensure that any user defined CLASSPATH variables are not used on startup,
# but allow them to be specified in setenv.sh, in rare case when it is needed.
CLASSPATH=
if [ -r "$CATALINA_BASE/bin/setenv.sh" ]; then
  . "$CATALINA_BASE/bin/setenv.sh"
elif [ -r "$CATALINA_HOME/bin/setenv.sh" ]; then
  . "$CATALINA_HOME/bin/setenv.sh"
fi
```