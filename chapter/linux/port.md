# 端口管理

## 端口增加
```bash
iptables -I INPUT -p tcp --dport 9101 -j ACCEPT
service iptables save
service iptables restart 
```