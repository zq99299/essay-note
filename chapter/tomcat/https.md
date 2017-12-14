# 开启HTTPS

修改tomcat目录下的 config.server.xml 
```xml
 // 开启https
 <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
                URIEncoding="utf-8"
                scheme="https"
                secure="true"
                SSLEnabled="true"
                keystoreFile="/data/xxx.keystore"
                keystorePass="xxxx"
                SSLCertificateFile="/data/xxx.crt"
                SSLCertificateKeyFile="/data/xxx.key"
                connectionTimeout="20000"
                redirectPort="8443" 
                compression="on"
                compressionMinSize="2048"
                noCompressionUserAgents="gozilla,traviata"
                compressableMimeType="text/html,text/xml,text/javascript,application/x-javascript,application/javascript,text/css,text/plain"
        />
 // 同时对该项目映射8080的http
 <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" 
               compression="on"
               compressionMinSize="2048"
               noCompressionUserAgents="gozilla,traviata"
               compressableMimeType="text/html,text/xml,text/javascript,application/x-javascript,application/javascript,text/css,text/plain"
        />

```

开启https需要以下几个字段的内容

```xml
scheme="https"
secure="true"
SSLEnabled="true"
keystoreFile="/data/xxx.keystore"   // 这个文件一定要有，否则tomcat7启动报错
keystorePass="xxxx"
SSLCertificateFile="/data/xxx.crt"
SSLCertificateKeyFile="/data/xxx.key"

```

关于证书的生成，可以百度

