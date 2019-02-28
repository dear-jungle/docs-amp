### 本地yum源搭建

> 离线yum源一般部署在管理节点服务器，一下操作均在管理节点服务器进行

#### **获取yum仓库包并创建本地yum源**

##### 1.在服务器上创建目录

```
# mkdir -p /var/www/html/
```

##### 2.解压yum.tar

```
# cd  /var/www/html/
# tar -zxvf CentOS.tar.gz
```

##### 3.配置本地yum源
关闭selinux,将SELINUX设置为disabled
```
setenforce 0
vi /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

vi /etc/selinux/config

离线环境需要先移除系统yum源配置
```
mkdir -p /etc/yum.repo/bak
mv /etc/yum.repo/Centos* /etc/yum.repo/bak/
```

```
# vi /etc/yum.repos.d/local.repo
//添加如下内容，请把IP替换为本机IP
[local]
name=local_repo
baseurl=http://IP/CentOS7.2/
enabled=1
gpgcheck=0
```

#### **安装配置相关软件**

##### 1.安装相关软件

```
# yum -y install httpd
# yum -y install createrepo
# yum install -y yum-plugin-priorities
```

##### 2.配置httpd

修改httpd.conf中的Listen端口以免端口冲突

```
# vi /etc/httpd/conf/httpd.conf 
//修改httpd服务端口为20000
Listen 20000
ServerName 20000
```

##### 3.启动

```
//开启httpd服务
# systemctl start httpd.service
//设置httpd自动启动
# systemctl enable httpd.service
```

##### 4.在仓库中新放置rpm包（可选）

> ```
> 必要的安装包都已经内置，如果需要加入新的RPM包可以执行下面步骤
> ```

```
//创建存放rpm包的路径。一定是两层级xxx/packages，xxx可以自己命名
# cd /var/www/html/CentOS7.2/repository
# mkdir -p xxx/packages
//将需要的rpm包放到xxx/packages目录下后，使包生效
# ./build_repo.sh
```

#### **验证**

浏览器访问，输入`http://172.16.76.6:20000/c2cloud_repo/`，其中`172.16.76.6`为仓库安装服务器ip

#### **客户端配置**

在需要用到yum仓库的节点配置

`/etc/yum.repos.d/`目录下配置`c2cloud`仓库配置文件：

```
# vi local.repo 
[local]
name=local_repo
baseurl=http://【YumHostIP】:20000/CentOS7.2/
enabled=1
gpgcheck=0
```

> ```
> 以上是文件编辑内容
>
> 第一行：yum仓库名字，任意。
>
> 第二行：详细名字，任意。
>
> 第三行：仓库路径。
>
> enabled为是否启用该仓库
> baseurl为关键字，为本地创建的yum仓库服务地址
>
> 第四行：关闭rpm包的gpg校验功能。如果个人环境，建议关闭，参数值为0，如果生产环境，建议打开，参数值为1
>
> 最后保存退出。
> ```

配置完成后清除域名仓库缓存

```
# yum makecache
```



