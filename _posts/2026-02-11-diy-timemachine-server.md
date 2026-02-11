---
layout: post
title:  "自建 timemachine 服务器"
date:   2026-02-11 21:08:00 +0800
categories: macos
tags: macos timemachine 备份 netatalk afp
author: 来财
---

直接说：之前是定期备份，每次备份就要插上移动硬盘，非常不方便。另外网络备份，如果选用 apple 的设备，那价格太感人了用不起。因此选择自建，局域网内同步除了首次同步需要耗费很长时间，其他还好。

* content
{:toc}





# 自建 timemachine 服务器

## 为什么要自建

直接说：之前是定期备份，每次备份就要插上移动硬盘，非常不方便。另外网络备份，如果选用 apple 的设备，那价格太感人了用不起。因此选择自建，局域网内同步除了首次同步需要耗费很长时间，其他还好。

## 创建备份用户

```shell

useradd tmbackup
passwd tmbackup

```

## 创建备份目录

`mkdir -p /mac/timemachine`

## netatalk安装

1. 安装必要的包

``` sudo yum install libdb-devel avahi-devel libacl-devel dbus-glib-devel ```

centos7,可能有改动，我这里就把所有可能的包都安装上

``` 
yum install epel-release -y
yum install avahi-devel bison cracklib-devel db4-devel dbus-devel dbus-glib-devel docbook-style-xsl flex krb5-devel libacl-devel libattr-devel libevent-devel libgcrypt-devel libtdb-devel libxslt nss-mdns mysql-devel openldap-devel openssl-devel pam_afs_session pam-devel pam_oath perl-IO-Socket-INET6 quota-devel systemtap-sdt-devel tcp_wrappers-devel -y

#Centos7装下面的，包有所改动
#yum install avahi-devel bison cracklib-devel dbus-devel dbus-glib-devel docbook-style-xsl flex krb5-devel libacl-devel libattr-devel libdb-devel libevent-devel libgcrypt-devel libtdb-devel libxslt nss-mdns mysql-devel openldap-devel openssl-devel pam_afs_session pam-devel pam_oath perl-IO-Socket-INET6 quota-devel systemtap-sdt-devel tcp_wrappers-devel tracker-devel -y
```

2. 下载 netatalk 亲测可用

```shell
wget https://nchc.dl.sourceforge.net/project/netatalk/netatalk/3.1.11/netatalk-3.1.11.tar.bz2
tar -xvf netatalk-3.1.11.tar.bz2 && cd netatalk-3.1.11/
./configure --with-init-style=redhat-systemd --with-acls --with-pam-confdir=/etc/pam.d --with-afpstats --with-dbus-sysconf-dir=/etc/dbus-1/system.d
make
sudo make install
mkdir ~/timemachine
```

备用

```shell
yum install gcc make rpm-build -y
cd /opt
wget http://www003.upp.so-net.ne.jp/hat/files/netatalk-3.1.11-0.1.1.fc27.src.rpm
rpm -ivh netatalk-3.1.11-0.1.1.fc27.src.rpm
cd ~/rpmbuild/SPECS/
rpmbuild -bb netatalk.spec
yum install /root/rpmbuild/RPMS/x86_64/netatalk-3.1.11-0.1.1.el6.x86_64.rpm
```

3. 编辑配置文件（==亲测可用==）

==如果是yum安装的netatalk==，配置文件在`vim /etc/netatalk/afp.conf `

``` 
sudo vim /usr/local/etc/afp.conf

Global]
; Global server settings
mimic model = TimeCapsul
log level = default:warn
log file = /var/log/afpd.log
;hosts allow = 192.168.1.0/24  #允许访问的主机地址
uam list = uams_clrtxt.so uams_dhx.so uams_dhx2.so uams_guest.so  #必须，认证方式，目前只调通了guest模式
guest account = nas  #必须，guest对应的linux系统用户

[TimeMachine]
path = /www/timemachine
time machine = yes  #必须，yes才支持mac timemachine
rwlist = tmbackup   #必须，设置nas 读写权限
force user = tmbackup  #必须，用户映射
vol size limit = 100000 #限制贡献volume大小为100GB，单位为MB
```

需要注意，配置文件中的用户是Linux的用户，所以说，请确保你的系统有这个用户且设置了密码，访问的时候会有认证

**另外，如果出现了无法访问和写入之类的问题，建议看下所有者和权限(建议把TM目录的所有者改成你指定的用户)**

PS.如果要允许匿名访问，可以在uam list处添加uams_guest.so，当然这个是只读的，如果要匿名读写（**不安全**），请修改为rwlist = nobody

这样就配置完Netatalk了，**接下来的Avahi看你需求，如果是在局域网内，那就配，不在局域网内不用开着，因为这个是用于服务自动发现的。**

### 安装Avahi

```
#一般centos7系统都自带
yum -y install avahi-daemon
yum -y install avahi
```

编辑开机启动文件配置 `vim /etc/avahi/services/afpd.service`

```
<service-group>
  <name replace-wildcards="yes">%h</name>
  <service>
    <type>_afpovertcp._tcp</type>
    <port>548</port>
  </service>
  <service>
    <type>_device-info._tcp</type>
    <port>0</port>
    <txt-record>model=Xserve</txt-record>
  </service>
</service-group>
```

```shell
#亲测
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">%h</name>
  <service>
    <type>_afpovertcp._tcp</type>
    <port>548</port>
  </service>
  <service>
    <type>_device-info._tcp</type>
    <port>0</port>
    <txt-record>model=TimeCapsule</txt-record>
  </service>
</service-group>
```

然后这时候我们就完成配置咯。下面启动服务

```
#centos7系统,亲测
systemctl start avahi
systemctl start netatalk
systemctl enable avahi-daemon
systemctl enable netatalk
```

```
service netatalk start
service avahi-daemon start
```

### 开机启动

```shell
#centos6
chkconfig netatalk on
chkconfig avahi-daemon on
```

| 然后就好咯，如果是局域网，而且配置了Avahi，那么你的Mac的Finder里应该过一会儿就会自动出现TimeMachine

不是的话你可能需要自己连接了

打开 Finder -> Go -> Connect to Server… 填入机器的 IP 地址（afp://192.168.1.11）后点击 Connect，使用刚才创建的帐号和密码登录进入之后就会看到 ==TimeMachine== 文件夹。

打开 System Preferences -> Time Machine -> On 选择 ==TimeMachine== 文件夹，然后点击 Use Disk 就应该能用了。

然后创建目录并设置权限

``` 
mkdir /home/timemachine
chown tmbackup:tmbackup /home/timemachine
```

4. 设置为开机自启动

``` systemctl enable netatalk ```

5. 启动 netatalk

``` systemctl start netatalk ```

# 使用 timemachine

# Linux AFP协议(Netatalk)配置

Linux的AFP协议（即Apple的文件共享协议）实际就是安装了一个软件Netatalk和Avahi。

Netatalk是一个开源的 Apple Talk 通信协议组，它允许类Unix 系统为 Mac 做文件服务器，打印服务器等等。

Avahi 是 Apple's Zeroconf 协议的开源实现，实现类似 Bonjour 的功能，它可以让你在 Mac 系统里自动发现你的 Linux 计算机。

其配置文件在`/etc/netatalk/AppleVolumes.default`或`/usr/local/etc/afp.conf`（WD MyCloud 在/etc/nas/afp_share.conf）

WD MyCloud的配置文件是服务启动时自动填的，真正需要改的是/etc/trustees.conf, 包括/etc/nas/apache2/auth/require.inc也是自动填的，只要这个trustees.conf中的用户权限是对的就可以。RWBEX代表所有权限，RBE代表只读权限。

一个目录也可以供多个用户使用，书写格式为[/dev/sda4]/shares:user1:RBE:user2:RWBEX:*:CU，这样表示/shares目录可以供user1和user2两个用户分别以只读和读写权限访问。

重启服务

```shell
service netatalk restart
service avahi-daemon restart
```