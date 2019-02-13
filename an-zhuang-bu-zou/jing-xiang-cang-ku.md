### 镜像仓库安装

> 参考“镜像仓库安装”章节部署的镜像仓库需要重启docker进程以实现https认证。本章与“镜像仓库安装”章节在生成harbor https证书上有所不同，且docker login时不需要重启docker进程就可以实现镜像仓库的https认证。

#### 安装前准备

安装镜像仓库之前需要确认：

* 已经安装docker环境，docker安装过程[参见这里](/an-zhuang-bu-zou/docker1131an-zhuang.md)
* 已经安装docker-compose
* harbor的存储目录（默认/data）有足够的硬盘空间
* 已经关闭防火墙和selinux

安装所需资源：docker.tar  harbor.tar

#### **安装docker-compose**

```
#解压harbor.tar拷贝docker-compose文件，并增加可执行权限
$ cp docker-compose /usr/local/bin/
$ chmod +x /usr/local/bin/docker-compose
```

##### 1.docker ip修改（可选）

> docker安装完成后，默认分配的网段是172.18.0.0/16 为了避免和宿主机局域网段冲突，可以对其进行修改。这里按照示例安装规划进行配置,添加bip属性 --bip=192.168.1.4/16

```
$ vi /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd $OPTIONS \
  --bip=192.168.1.4/16
```

##### 2.启动docker

```
$ sudo systemctl enable docker
$ sudo systemctl  start docker
```

#### **安装harbor V1.0**

```
$ tar -zxvf  harbor.tgz
$ cd harbor
#修改配置
$ mv docker-compose.yml docker-compose.yaml.bak
$ cp /opt/harbor/docker-compose.yml docker-compose.yaml
```

#### **为harbor配置HTTPS安全访问（可选）**

##### 1. 参考[openssl安装文档](https://mritd.me/2016/07/03/Harbor-企业级-Docker-Registry-HTTPS配置/#11san-证书扩展域名配置)安装openssl服务

##### 2. 获得根证书文件

```
# openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout ca.key \
    -x509 -days 3650 -out ca.crt \
    -subj "/C=CN/ST=hunan/L=cs/O=kc/OU=IT/CN=kc/emailAddress=kc@hnkc.com"

# ls
ca.crt  ca.key
```

**3. 生成证书签名请求**

```
# openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout docker.key \
    -out docker.csr \
    -subj "/C=CN/ST=hunan/L=cs/O=kc/OU=IT/CN=HARBOR_IP/emailAddress=kc@hnkc.com"

# ls
ca.crt  ca.key  docker.csr  docker.key
```

** 4. 为registry产生证书**

```
# echo subjectAltName = IP:192.168.69.128 > extfile.cnf

# openssl x509 -req -days 365 -in docker.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out docker.crt

# ls
ca.crt  ca.key  ca.srl  extfile.cnf  docker.crt  docker.csr  docker.key
```

##### 

##### 5. 服务端配置相对简单，只需要修改一下 Harbor 的 Nginx 配置文件，并把签名好的证书和私钥复制过去即可

```
# 复制 crt、key
$ cp ~/dockercrt/docker.crt /data/cert/
$ cp ~/dockercrt/docker.key /data/cert/
```

##### 6. 修改harbor证书认证方式

```
# cd /root/harbor/
# vi harbor.cfg
//只需要修改如下配置项
hostname = your-ip    ##harbor的机器IP
ui_url_protocol = https    ##harbor的认证方式
ssl_cert = /data/cert/docker.crt 
ssl_cert_key = /data/cert/docker.key   ##harbor证书位置
```

##### 7.重启镜像仓库

```
$ cd /root/harbor
$ ./prepare                     ##新安装或者修改了配置文件需要执行
$ docker-compose down
$ docker-compose up -d
```

#### 检查是否安装成功

访问镜像仓库地址\(因为默认开放80和433端口，直接访问ip即可\)

[http://ip](http://ip) 或者 [https://ip](https://ip)

如登陆镜像仓库：172.17.87.19 ，账号/密码：admin/Harbor12345

能正常访问说明镜像仓库安装成功

> 如果是https访问，因为是自签名证书，需要点击页面上的`高级-`\[`继续前往`\]

#### 初始化镜像仓库

登录系统，点击界面上的项目-新增项目，新增两个项目 admin，c2cloud并且设置为公开。

#### 设置docker可访问镜像仓库

1. 去掉docker配置文件中的**--insecure-registry**配置（如果修改了配置文件需要重启docker）
2. 新建目录
   ```
   mkdir /etc/docker/certs.d/{HARBORIP}
   ```
3. 将ca.crt拷贝到新建的目录下
   ```
   # scp ca.crt root@172.17.80.88:/etc/docker/certs.d/{harborip}/
   ```
4. 登录验证
   ```
    docker login {harborip}
   ```



