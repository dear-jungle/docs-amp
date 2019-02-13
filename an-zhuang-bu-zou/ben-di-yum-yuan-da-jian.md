### 本地yum源搭建

> 以下操作均在yum安装服务器执行

#### **获取yum仓库包并创建本地yum源**

##### 1.在服务器上创建目录

```
# mkdir /opt/c2cloud
```

##### 2.解压yum.tar

```
# cd  /opt/c2cloud
# tar -zxvf repository.tar.gz
```

##### 3.配置本地yum源

```
# vi /etc/yum.repos.d/local.repo
//添加如下内容
[tools] 
name=tools
baseurl=file:/opt/c2cloud/repository/
enabled=1 
gpgcheck=0 
priority=1
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
//修改httpd服务端口为8090
Listen 8090
```

新建仓库访问地址

```
# vi /etc/httpd/conf.d/c2cloud_repo.conf 
//创建一个c2cloud_repo.conf文件，内容如下：

Listen 20000
<VirtualHost *:20000>
   Alias /c2cloud_repo /opt/c2cloud/repository
   <Directory /opt/c2cloud/repository>
      Options Indexes MultiViews FollowSymLinks
      Allow from all
      AllowOverride None
      Require all granted
   </Directory>
</VirtualHost>
```

##### 3.启动

```
//开启httpd服务
# systemctl start httpd.service
//设置httpd自动启动
# systemctl enable httpd.service
```

##### 4.在仓库中新放置rpm包

```
//创建存放rpm包的路径。一定是两层级xxx/packages，xxx可以自己命名
# cd /opt/c2cloud/repository
# mkdir xxx
# mkdir xxx/packages
//将需要的rpm包放到xxx/packages目录下后，使包生效
# cd /opt/c2cloud/repository/
# ./build_repo.sh
```

#### **验证**

浏览器访问，输入`http://172.16.76.6:20000/c2cloud_repo/`，其中`172.16.76.6`为仓库安装服务器ip

#### **客户端配置**

在需要用到yum仓库的节点配置

`/etc/yum.repos.d/`目录下配置`c2cloud`仓库配置文件：

```
# vi c2cloud.repo 
[c2cloud]
name = c2cloud
baseurl = http://172.16.76.6:20000/c2cloud_repo/   ##根据实际的yum仓库地址进行修改
enabled = 1
gpgcheck = 0
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
> baseurl为关键字，为本地创建的镜像仓库服务地址
>
> 第四行：关闭rpm包的gpg校验功能。如果个人环境，建议关闭，参数值为0，如果生产环境，建议打开，参数值为1
>
> 最后保存退出。
> ```

配置完成后清除域名仓库缓存

```
# yum clean all
```



