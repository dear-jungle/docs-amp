### docker安装

#### 安装前准备

* 安装主机操作系统必须是CentOS Linux release 7.4/7.5，内核版本必须3.10.0-514.el7.x86\_64及以上。

* 安装docker之前需要完成本地yum源搭建。

* 安装所需资源：本地yum源，初始化配置脚本install-docker.sh。

#### docker离线安装

```
$  sh install-docker.sh
```

> 此方式为离线yum安装，如何安装离线yum源，请参见[本地yum源搭建](/an-zhuang-bu-zou/ben-di-yum-yuan-da-jian.md)

##### docker虚拟ip修改（可选）

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
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:23:03 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:25:29 2018
  OS/Arch:          linux/amd64
  Experimental:     false
```

2.检查docker存储驱动配置正确

```
# docker info
//有如下信息表示存储配置正确
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
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



