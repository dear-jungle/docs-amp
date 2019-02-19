### 应用管理门户部署

我们提供两种部署应用集成平台方式：k8s部署和docker部署

k8s部署：基于kubernetes集群，可横向扩展，可以搭配应用部署平台实现应用的快速部署和集成，同时提供应用的监控，日志，端口管理，伸缩和迁移以及版本的无缝升级等功能，为应用日常运维提供了非常丰富快捷的管理功能。

docker部署:对部署环境要求不高，只要安装docker环境就可以，但后续的应用部署都要通过docker命令行操作，无法监控管理应用的生命周期，不提供应用的编排，应用之间的通信需要手动配置，对运维的要求较高。

部署者可以根据实际情况选择基于kubernetes或者docker中的一种部署应用集成平台。

本章介绍如何在kubernetes上部署应用集成平台，_以下操作在kubernetes的管理节点执行！_

#### 安装前准备

安装前请确认：

* 已经安装docker环境
* 已经安装kubernetes环境
* 已经部署oracle服务,且集群可访问

安装需要资源：aip.tar

### 需要最低配置

运行应用集成平台需要集群拥有最少8G空闲内存，20G空闲磁盘存储空间

#### **安装步骤**

##### 1.登录到kubernetes管理节点

##### 2.解压aip.tar

```
$ cd /opt
$ tar -zxvf aip.tar
$ cd aip
```

##### 3.选择一个计算节点运行应用集成平台容器\(假设选择的计算节点的hostname=172-17-80-40.node\)

```
$ kubectl labels node 172-17-80-40.node cluster=admin tenant=admin
$ kubectl get node --show-labels
```

设置成功将在LABELS下包含：cluster=admin，tenant=admin

```
NAME                  STATUS         AGE       LABELS

172-17-80-39.master   Ready,master   88d       beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,cluster=admin,kubeadm.alpha.
```

##### 4.登录172-17-80-40.node，解压镜像

```
#这步在172-17-80-40.node机器上执行，aip-images.tar可以在解压aip.tar后找到
$ docker load -i aip-images.tar
```

##### 5.配置清单，根据本地环境修改环境变量

```
$ vi k8s-script/config
```

以下为config模板，仅供参考，具体配置需要根据本地环境修改

```
#想把应用部署到哪个命名空间
namespace=admin
#部署到哪个集群
cluster=default
#nfs根目录
nfs_root_path=/data/volumes
#nfs服务器ip
nfs_server_ip=10.0.203.26
#数据库配置
aip_db_url=jdbc:oracle:thin:@10.0.203.17:1521:orclpdb
aip_db_username=YXX_CEP_AIP
aip_db_password=888888
uop_db_url=jdbc:oracle:thin:@10.0.203.17:1521:orclpdb
uop_db_username=YXX_CEP_UOP
uop_db_password=888888
db_url=jdbc:oracle:thin:@10.0.203.17:1521:orclpdb
db_username=YXX_CEP_DICT
db_password=888888
#统一机构用户平台外部地址
uop_sso_client_clienturl=http://10.0.203.24:30008
#统一认证授权服务器地址,即应用集成平台地址
sso_client_authrizationserverurl=http://10.0.203.24:30009
#应用集成平台镜像
integration_image=registry.c2cloud.cn/c2cloud/application-integration-platform:v2.1.0                                                    
#统一机构用户平台镜像
uop_platform_image=registry.c2cloud.cn/c2cloud/user-organization-applications:v2.0.3.8
#统一机构用户服务镜像
uop_service_image=registry.c2cloud.cn/c2cloud/unify-user-organization-services:v2.0.3.10
#统一字典服务镜像
dict_images=registry.c2cloud.cn/c2cloud/dict-server:v1.1
```

##### 6.sql初始化

* 新建XXX\__CEP\_AIP、XXX\_CEP\_UOP、XXX\_CEP\_DICT三个用户，把相关sql导入oralce12c_
* 服务网关容器启动后，找到网关的postgres数据库对应的端口号，初始用户名密码为kong/kong，删除kong用户下的表和数据，导入网关postgres.sql

```
   $kubectl get svc -n=admin
```

##### ![](/assets/import2.png)

##### ![](/assets/import3.png)

##### 7.生成部署脚本

```
$ chmod +x build.sh
$ ./build.sh
```

##### 8.运行容器

```
$ kubectl create namespace admin
$ kubectl create -f k8s-script.yaml
```

#### **验证**

访问应用集成平台

[http://master-ip:30009](http://master-ip:30009)

账户/密码 admin/admin

