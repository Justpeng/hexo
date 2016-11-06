---
title: Hadoop分布式集群搭建 Centos7 环境准备
date: 2016-10-22 17:58:27
description: "Hadoop分布式集群搭建"
categories: [Hadoop]
tags: [Hadoop]
top: 13
---

## 机器准备

- Centos7 
- Vmware10
- jdk

### 安装和配置IP

主机名称 |用户|密码| ip
---|---|---|---
master |master|master| 192.168.1.122
slave1 | hadoop|hadoop|192.168.1.123
slave2 |hadoop|hadoop|192.168.1.124
slave3 |hadoop |hadoop|192.168.1.125

在每台主机添加用户 hadoop 密码 hadoop

>使用命令 useradd添加用户， passwd为用户hadoop设置密码

```
[root@slave1 ~]# useradd hadoop
[root@slave1 ~]# passwd hadoop
Changing password for user hadoop.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@slave1 ~]# 

```

### 关闭防火墙
关闭防火墙是必须的,centos7默认是不用iptables

```
sudo systemctl stop firewalld.service && sudo  systemctl disable firewalld.service
```

### 配置hosts文件
注意： ***==sudo==*** 命名为使用超级管理员用户操作 相当于 root 用户

```
[hadoop@localhost ~]$ sudo vim /etc/hosts

```
问题：若无法使用sudo命令？
```
hadoop is not in the sudoers file.  This incident will be reported.

```
需要在 /etc/sudoers文件中添加配置

```
## Allow root to run any commands anywhere 
root	ALL=(ALL) 	ALL
hadoop ALL=(ALL) ALL

```
hosts文件中添加内容为(其他主机同样添加)：
```
192.168.1.122 master
192.168.1.123 slave1
192.168.1.124 slave2
192.168.1.125 slave3

```
## 配置SSH 免登陆

这里需要对 所有机器上的 ==hadoop@主机== 用户配置ssh免密码登录

在Centos7中已经默认安装ssh服务，ssh-keygen命令无效，说明需要先安装ssh服务,即：
```
[hadoop@master /]$ sudo yum install openssh-server
```
### 以==master==主机为例.
- 生成无密码密钥对,在命令中一路回车即可
```
[hadoop@master /]$ ssh-keygen
```

- ### 将公钥写入公钥库中,这里公钥库使用默认库名 authorized_keys

```
[hadoop@master /]$  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

```
查看.ssh中内容
```
[hadoop@master ~]$ cd ~/.ssh
[hadoop@master .ssh]$ ls
authorized_keys  id_rsa  id_rsa.pub  known_hosts

```

- 设置公钥库权限
```
[hadoop@master /]$ chmod 600 authorized_keys 
```
- 重启ssh服务 (root权限)

```
[hadoop@master /]$ sudo service sshd restart

```
**其他主机，即slave1,slave2,slave3，分别执行 ssh-keygen生成密钥对**

- 将maser主机中的ssh分别发至 slave1,slave2,slave3主机

其中 slave1@slave1 即 xx@yy xx代表用户名，yy代表主机名
```
[hadoop@master /]$ scp ~/.ssh/* hadoop@slave1:~/.ssh/
[hadoop@master /]$ scp ~/.ssh/* hadoop@slave2:~/.ssh/
[hadoop@master /]$ scp ~/.ssh/* hadoop@slave3:~/.ssh/

```
重启其他主机的ssh服务

- 测试

主机之间第一次访问时，会需要接受ESDSA可以，yes接受即可，再次互相访问时，就就不在需要了。


```
[hadoop@slave3 ~]$ ssh master
The authenticity of host 'master (192.168.1.122)' can't be established.
ECDSA key fingerprint is 3b:14:33:d5:5a:c4:92:c5:62:65:49:3d:4f:90:07:16.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'master,192.168.1.122' (ECDSA) to the list of known hosts.
Last login: Sun Nov  6 06:03:47 2016 from 192.168.1.105
[hadoop@master ~]$ ssh slave1
Last login: Sun Nov  6 06:05:12 2016 from 192.168.1.105
[hadoop@slave1 ~]$ ssh slave2
Last login: Sun Nov  6 06:11:48 2016 from 192.168.1.105
[hadoop@slave2 ~]$ ssh slave3

```

maser,slave1,slave2,slave3 四台主机之间都可以无密码访问，即是ssh设置成功了！

## 配置JDK
这里对==master==主机进行配置 将jdk等软件上传至/opt/目录：

- 使用*超级管理员*用户解压 jdk

```
sudo tar -zxvf jdk-7u67-linux-x64.tar.gz 
```
配置jdk环境变量 /etc/profile，并查看是否成功
```
export JAVA_HOME=/opt/jdk1.7.0_67/
export JRE_HOME=/opt/jdk1.7.0_67/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

```

注意，上面配置中，最后一行，$PATH放在前面不生效时，也可以这么写：
```
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin：$PATH

```

重新加载

```
source /etc/profile
```



