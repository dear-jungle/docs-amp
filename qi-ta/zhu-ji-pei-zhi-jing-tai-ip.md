### 主机配置静态IP

在集群所有节点执行以下操作

* **网络配置**

1.配置静态IP

```
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
//修改BOOTPROTO为static
BOOTPROTO=static
//添加静态IP，网关，广播地址，子网掩码,根据自己的实际网络规划来
BROADCAST=172.17.87.255
IPADDR=172.17.87.35
NETMASK=255.255.255.0
GATEWAY=172.17.87.254
```

> 该操作主要针对IP为动态获取的情况，如果已经静态配置IP则不需要做此操作



