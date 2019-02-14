### nfs存储服务部署

#### 安装前准备

安装nfs前请确认：

* 已经给nfs根目录分配足够的硬盘存储空间
* 已经安装本地yum仓库，或者可以访问外网

**nfs服务端部署**

```
#本地yum源安装nfs
$ yum -y install nfs-utils --enablerepo=c2cloud
$ vi /etc/exports
#添加如下内容 /data为nfs存储根路径
/data    *(rw,no_root_squash,sync,subtree_check)
$ systemctl enable nfs
$ systemctl start nfs
```

**nfs客户端安装\(可选\)**

安装nfs客户端命令：

```
$ yum -y install nfs-utils --enablerepo=tools
```

> **用户可以根据实际需求选择合适的持久化存储方式，可选项：ceph rbd，nfs，hostpath**



