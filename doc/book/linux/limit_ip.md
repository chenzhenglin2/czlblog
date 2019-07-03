# 如何利用iptables或防火墙指定ip访问服务器某个端口

## 有如下两种方式：

### 1

```bash
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=10.219.82.83 port port=12181 protocol=tcp accept'

systemctl  restart  firewalld
```



移除--add-rich-rule规则  用--remove-rich-rule=

帮助文档显示：

```
  --list-rich-rules    List rich language rules added for a zone [P] [Z]

  --add-rich-rule=<rule>

​                       Add rich language rule 'rule' for a zone [P] [Z] [T]

  --remove-rich-rule=<rule>

​                       Remove rich language rule 'rule' from a zone [P] [Z]
```



### 2

```
iptables-save > iptables.rules

/sbin/iptables -A INPUT -s 172.16.35.31 -p tcp --dport 12181 -j ACCEPT          # 中控白名单

/sbin/iptables -A OUTPUT -d 172.16.35.31 -p tcp --sport 12181 -j ACCEPT

/sbin/iptables -A INPUT -p tcp --dport 12181 -j DROP   #其他服务器无法访问12181端口

/sbin/iptables -A OUTPUT -p tcp --sport 12181 -j DROP

\#iptables --list-rules  #查看规则 ，根据情况删除下面规则

\#iptables -D INPUT -j REJECT --reject-with icmp-host-prohibited   #

\#iptables -D FORWARD -j REJECT --reject-with icmp-host-prohibited  #

service  iptables  save

systemctl  restart   iptables
```

