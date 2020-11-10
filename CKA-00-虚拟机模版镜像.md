安装VMware Fusion


讲师的环境是windows10，虚拟机是vmwarevm，给到是一个k8s-tmp目录，而我使用到是Mac OS的VMware Fusion。

将整个目录（k8s-tmp）的目录名加上后缀 .vmwarevm，双击 k8s-tmp.vmwarevm 使用 VMware Fusion 直接打开即可。

启动k8s-tmp



远程连接
```shell
$ ssh root@172.16.99.129
```
登录名：root
密码：redhat

有些镜像是需要光驱的，有些则不需要，该模版镜像士不需要光驱，删除即可（已经删除）

centos 版本：
```shell
$ cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core) 
```

挂在磁盘情况:
```shell
$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk 
├─sda1   8:1    0  150G  0 part /
└─sda2   8:2    0   10G  0 part [SWAP]
```
总共划分了200G空间,虽然划分了200G,但是并不真正占用宿主机的200G存储,而是用多少是多少，包含两个分区，sda1 挂载是根目录，sda2 挂载是swap ，但是在kubernetes中是不能使用swap的，所以需要关闭swap。

关闭swap:
```shell
$ vim /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Thu Oct 18 23:09:54 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=9875fa5e-2eea-4fcc-a83e-5528c7d0f6a5 /                       xfs     defaults        0 0
#UUID=16eb815a-8221-4e2e-8bed-c2e74bd2368e swap                    swap    defaults        0 0
```
> 注释掉swap 条目

修改网卡信息，修改成如下设置，其他删除即可：
```shell
$ cat /etc/sysconfig/network-scripts/ifcfg-ens32 
TYPE=Ethernet
BOOTPROTO=dhcp
NAME=ens32
DEVICE=ens32
ONBOOT=yes
```

修改yum 源：
```shell
$ yum install wget -y

$ cd /etc/yum.repos.d/
$ rm -rf ./*

$ wget ftp://ftp.rhce.cc/k8s/*
```

关闭selinx，设置内容成如下格式:
```shell
$ vim /etc/selinux/config

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

设置防火墙策略，任何数据包都可信任：
```shell
$ firewall-cmd --set-default-zone=trusted
Warning: ZONE_ALREADY_SET: trusted
success
```

设置命令行样式,增加PS1 参数：
```shell
$ vim /etc/bashrc

# /etc/bashrc
PS1='\[\e[32;48m\][\u@\h \W]\$ \[\e[m\]'

# System wide functions and aliases
# Environment stuff goes in /etc/profile
...
...
...

```
写入/etc/bashrc 文件方式为永久生效，退出shell，重新登录见效果。
或者使用下面的命令使配置文件生效：
```shell
$ source /etc/bashrc
```

安装bash-completion：
```shell
$ yum install bash-co*
```

因为是模版
```shell
$ rm -rf /etc/ssh/ssh_host_*
```

提升ssh 连接速度
```shell
$ vim /etc/ssh/sshd_config
UseDNS no
```

清空machine-id 文件内容，否则克隆出来的机器的machine-id相同。
```shell
$ cat /dev/null > /etc/machine-id
```

设置特定ip和主机名脚本，克隆后使用
```ssh
[root@localhost ~]# cat set.sh 
#!/bin/bash
if [ $# -eq 0 ]; then
	echo "usage: `basename $0` num"
	exit 1
fi
[[ $1 =~ ^[0-9]+$ ]]
if [ $? -ne 0 ]; then
	echo "usage: `basename $0` 10~240"
	exit 1
fi

cat > /etc/sysconfig/network-scripts/ifcfg-ens32 <<EOF
TYPE=Ethernet
BOOTPROTO=none
NAME=ens32
DEVICE=ens32
ONBOOT=yes
IPADDR=172.16.99.${1}
NETMASK=255.255.255.0
GATEWAY=172.16.99.2
DNS1=172.16.99.2
EOF

systemctl restart network &> /dev/null
ip=$(ifconfig ens32 | awk '/inet /{print $2}')
sed -i '/172/d' /etc/issue
echo $ip
echo $ip >> /etc/issue
hostnamectl set-hostname vms${1}
```

Mac OS 宿主机网卡信息：
```shell
$ ifconfig
...
...
vmnet1: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	ether 00:50:56:c0:00:01 
	inet 192.168.98.1 netmask 0xffffff00 broadcast 192.168.98.255
vmnet8: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	ether 00:50:56:c0:00:08 
	inet 172.16.99.1 netmask 0xffffff00 broadcast 172.16.99.255
```


右击k8s-tmp --> 创建完成克隆...,选择指定目录，命名镜像名称，保存即可完成镜像克隆。

启动已经克隆完成的机器

关闭屏保，防止黑屏
```shell
$ setterm -blank 0
```
使用set.sh 脚本，设置特定ip 和主机名
```shell
$ ./set.sh 31
172.16.99.31
```

然后即可使用ssh远程连接至172.16.99.31