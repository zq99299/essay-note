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
