### docker安装

##### 无外网yum安装：

```
$  yum -y install docker-engine --enablerepo=c2cloud
```

> 此方式为离线yum安装，如何安装离线yum源，请参见本地yum仓库搭建

##### docker ip修改（可选）

> docker安装完成后， 用 $ ip route show 查看docker0网段，如果和宿主机局域网段冲突，可以对其进行修改。这里按照示例安装规划进行配置，例如，添加bip属性 --bip=10.1.40.1/16

```
$ vi /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd $OPTIONS \
  --bip=192.168.1.1/16
```

#### 设置docker可访问镜像仓库

1. 新建目录/etc/docker/certs.d/{HARBORIP}
2. 将安装镜像仓库主机上生成的ca.crt拷贝到新建的目录下
   ```
   # mkdir -p /etc/docker/certs.d/{harborip}
   # cp ca.crt /etc/docker/certs.d/{harborip}/
   ```


#### **安装完成后检查**

1.检查docker已安装且版本正确

```
# docker version
Client:
 Version:      1.13.1
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   092cba3
 Built:        Wed Feb  8 06:38:28 2017
 OS/Arch:      linux/amd64
```

2.检查docker存储驱动配置正确

```
# docker info
//有如下信息表示存储配置正确
Storage Driver: devicemapper
Pool Name: docker-thinpool
```

3.检查docker正常运行

```
$ systemctl status docker
#状态为running，说明正常运行
docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2017-08-07 19:56:43 CST; 1 weeks 6 days ago
     Docs: https://docs.docker.com
 Main PID: 1010 (dockerd)
   Memory: 349.0M
   CGroup: /system.slice/docker.service
```

4.检查镜像仓库访问

```
# docker login 镜像仓库-ip
Username: admin
Password:                       ##默认密码是Harbor12345 在/root/harbor/harbor.cfg中配置harbor_admin_password=Harbor12345
Login Succeeded
```



