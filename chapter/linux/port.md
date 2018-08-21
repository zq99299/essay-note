# 端口管理

## 端口增加
```bash
iptables -I INPUT -p tcp --dport 9101 -j ACCEPT
service iptables save
service iptables restart
```

## centos7

```
# 查看防火墙状态

firewall-cmd --state

# 停止firewall

systemctl stop firewalld.service

# 禁止firewall开机启动

systemctl disable firewalld.service
```

## 时区设置
CentOS7、RHEL7、Scientific Linux 7、Oracle Linux 7
```
最好的方法是使用timedatectl命令

# timedatectl list-timezones |grep Shanghai    #查找中国时区的完整名称
Asia/Shanghai
# timedatectl set-timezone Asia/Shanghai    #其他时区以此类推
```
