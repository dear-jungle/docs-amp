### kubernetes安装

#### 安装前准备

安装前请确认：

* 已经安装本地yum源
* 已经设置静态IP（[如何设置](/an-zhuang-bu-zou/lu-you-bu-shu.md)）
* 已经安装docker环境

安装kubernetes1.10.0所需资源：install.tar.gz

> kubernetes是部署应用集成平台的基础架构，有很多节点，可横向扩展。部署需要多台机器，且需要指定其中一台作为管理节点，其余作为计算节点，具体可以参见[示例规划](/an-zhuang-zhun-bei/shi-li-gui-fan.md)章节。

#### **部署管理\(master\)节点**

##### 1.解压install.tar.gz

```
$ cd /opt
$ tar -xzvf install.tar.gz
```

##### 2.执行intall-master.sh

```
$ cd install
$ sh new-install-master.sh ip   #ip为本机ip
```

##### 3.启动集群网络

```
$ kubectl create -f  kube-flannel.yml
```

##### 4.设置DNS

```
$ vi /etc/resolv.conf
search svc.cluster.local
nameserver 10.96.0.10
```

> 注意：如果有多个search和nameserver的情况下要归类放在一起。

#### **安装完成后检查**

##### 1.查看节点情况

```
//能查看到所有的master及node节点
[root@172-17-87-35 ~]# kubectl get node
NAME                  STATUS         AGE
172-17-87-35.master   Ready,master   12m
```

##### 2.查看创建的基础pod，其中STATUS都为running状态说明主节点部署成功

```
//能查看到以下的pod且STATUT均为Running表示k8s集群创建成功
[root@172-17-87-35 k8s]# kubectl get pod --namespace=kube-system
NAME                                          READY     STATUS    RESTARTS   AGE
dummy-2088944543-ldzh0                        1/1       Running   0          26m
etcd-172-17-87-35.master                      1/1       Running   36         24m
kube-apiserver-172-17-87-35.master            1/1       Running   33         24m
kube-controller-manager-172-17-87-35.master   1/1       Running   1          24m
kube-discovery-1769846148-m0c9c               1/1       Running   0          26m
kube-dns-2924299975-wkwxq                     4/4       Running   0          25m
kube-proxy-gt0vp                              1/1       Running   0          15m
kube-scheduler-172-17-87-35.master            1/1       Running   1          24m
kubernetes-dashboard-3109525988-1qsfn         1/1       Running   0          33s
weave-net-3d5h7                               2/2       Running   0          1m
```

##### 3.检查DNS是否正常

```
$ ping kubernetes.default 
# 如果可以正确解析出ip 说明DNS配置正常
PING kubernetes.default.svc.cluster.local (10.96.0.1) 56(84) bytes of data.
```

#### **部署计算（node）节点**

##### 1.解压install.tar.gz

```
$ cd /opt
$ tar -xzvf install.tar.gz
```

##### 2.在主节点上获取准入token

```
$  kubeadm token list
```

##### 3.执行intall-master.sh

```
$ cd install
#token 为第二步执行结果，ip为本机ip，master-ip为部署管理节点机器IP
$ sh  new-install-node.sh token ip master-ip
```

#### **安装完成后检查**

查看节点情况（在管理节点执行）

```
#172-17-87-36.node为计算节点的hostname，STATUS为ready状态说明成功加入到集群
[root@172-17-87-35 ~]# kubectl get node
NAME                  STATUS         AGE
172-17-87-35.master   Ready,master   12m
172-17-87-36.node     Ready          2m
```



