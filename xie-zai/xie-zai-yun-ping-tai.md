### 卸载云平台

#### 卸载docker

列出你安装过docker的包

```
# yum list installed | grep docker
docker-engine-selinux.noarch 1.12.3-1.el7.centos @dockerrepo
docker-io.x86_64 1.7.1-2.el6 @epel
```

删除安装包

```
# sudo yum -y remove docker-engine-selinux.noarch
```

```
# sudo yum -y remove docker-io.x86_64
```

删除镜像/容器等，如果需要保留这部分数据则可不删除

```
$ rm -rf /var/lib/docker
```

#### 卸载kubernetes

在已经安装kubernetes的节点，执行命令

```
$ kubeadm reset
#重启主机
$ shutdown -r now 
```



