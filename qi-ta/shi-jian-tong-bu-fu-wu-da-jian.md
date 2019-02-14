### 时间同步服务配置

#### **ntp服务端配置（master节点执行）**

##### 1.安装时间服务器ntp

```
# yum -y install --enablerepo=c2cloud  ntp
```

##### 2.备份配置文件

```
# mv /etc/ntp.conf /etc/ntp.conf.bak
```

##### 3.修改配置文件

```
# vi /etc/ntp.conf
//新增如下内容
#fast ntp server
server 127.127.1.0                              ##服务器同步的时间为本机
fudge 127.127.1.0 stratum 8


#store last time
driftfile /etc/ntp/drift

#allow upper modify localhost
restrict 0.0.0.0 nomodify notrap noquery

#allow any host
restrict 0.0.0.0 mask 0.0.0.0 nomodify notrap

#ntp log path
statsdir /var/log/ntp/

#ntp log file
logfile /var/log/ntp/ntp.log
```

##### 4.启动ntp服务端

```
//设置为开机启动
# systemctl  enable ntpd
//启动命令
# systemctl  start  ntpd
```

#### 检查服务是否正常

```
# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*LOCAL(0)        .LOCL.           8 l   21   64  377    0.000    0.000   0.000
```

#### **ntp客户端配置（node节点执行）**

##### 1.安装时间服务器ntp

```
# yum -y install   ntpdate
```

##### 2.初始同步时间

```
# ntpdate -u master-ip
21 Mar 11:21:55 ntpdate[1531]: adjust time server 172.17.87.35 offset 0.063120 sec
```

> ##### master-ip ntp服务端所在的ip

##### 3.定时同步设置

```
//在该配置文件增加如p下内容：
//ntp 每隔10分钟同步ntp服务端时间，同时把系统时间写入到硬件时间，your-ntpserver-ip替换成前面配置好的ntp服务端IP
# echo "*/10 * * * * root /usr/sbin/ntpdate master-ip; /sbin/hwclock -w" >> /etc/crontab
```

##### 4.重启定时任务

`$ service crond restart`

#### **无法成功同步时间原因分析**

1、客户端的日期必须要设置正确，不能超出正常时间24小时，不然会因为安全原因被拒绝更新。其次客户端的时区必须要设置好，以确保不会更新成其它时区的时间。

2、fudge 127.127.1.0 stratum 10 如果是LINUX做为NTP服务器，stratum\(层级\)的值不能太大，如果要向上级NTP更新可以设成2

3、LINUX的NTP服务器必须记得将从上级NTP更新的时间从系统时间写到硬件里去 hwclock --systohc

```
 NTP一般只会同步system clock. 但是如果我们也要同步RTC\(hwclock\)的话那么只需要把下面的选项打开就可以了
  代码:
  # vi /etc/sysconfig/ntpd
  SYNC\_HWCLOCK=yes
```

4、Linux如果开启了NTP服务，则不能手动运行ntpdate更新时间（会报端口被占用），它只能根据/etc/ntp.conf 里server 字段后的服务器地址按一定时间间隔自动向上级NTP服务器更新时间。可以运行命令 ntpstat 查看每次更新间隔如：

\[root@ESXI ~\]\# ntpstat

synchronised to NTP server \(210.72.145.44\) at stratum 2

本NTP服务器层次为2，已向210.72.145.44 NTP同步过

time correct to within 93 ms

\#时间校正到相差93ms之内

polling server every 1024

\#每1024秒会向上级NTP轮询更新一次时间

