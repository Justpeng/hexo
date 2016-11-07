---
title: Linux常用命令小结
date: 2016-11-07 19:58:27
description: "Linux常用命令"
categories: [Linux]
top: 15
---

环境：Centos7

### 查询如何使用命令

名称| 命令 | 示例
---|---|---
概述命令| whatis| whatis ls
命令作用、参数|man|  man ls
查看文件作用| man -k  |man -k /etc/hosts
查看命令作用|info| info grep

  
### 辅助命令

名称  | 命令 | 示例
---|---|---
列出最近使用的命令| history |
清屏 |clear |
查看当前位置|pwd | 在修改配置文件时，在shell中复制目录很方便
切换用户 |su - | su - root 切换至root用户，exit退出
使用超级管理员权限| sudo | sudo vim etc/profile
查看当前登录用户 | who|
查看当前用户 | whoami |

### 基本命令1

名称  | 命令 | 示例
---|---|---
 查看列表| ll/ls  | ls可直接查看操作权限等
 创建文件| touch | touch test.txt
 查看文件|more/cat/tail | tail -n 12 /etc/profile 表示从下向上读取12行
 编辑文件| vi / vim | vim /etc/profile,输入i进去编辑方式，esc退出编辑模式，wq保存退出等
  复制|cp | 常用参数：-r 目录递归复制,-f 强制复制等
 设置权限|chmod | chmod 600 test.txt 指读写等权限
 设置用户归属|chown | 将指定文件的拥有者改为指定的用户或组
 删除|rm | rm test.txt
  模糊匹配|grep |  ![image](http://og6e19zce.bkt.clouddn.com/linux_grep.png)



### 基本命令2
名称  | 命令 | 示例
---|---|---
输出文件的行数、字节数、单词数| wc  | wc test.txt， 参数：-l 行数，-c字节数，-w单词数，-L最长行的长度
当前目录下有多少普通文件和目录| ls\|wc |
当前有多少个进程|ps|wc|
建立软连接| ln -s|
编辑计划任务|crontab -e| * * * * * chmand :分 时 天 月 星期 命令
列出计划任务|crontab -l|


### 压缩包操作

后缀  | 命令 | 示例
---|---|---
tar.gz| tar | -zxvf 解压缩,-cxvf 压缩
zip | |unzip解压缩 ，zip 压缩
gzip|gzip| -d 解压缩，-r压缩

### 用户操作
名称  | 命令 | 示例
---|---|---
添加组|groupadd   | groupadd tgroup 添加组'tgroup',在 /etc/group文件中有组信息，若加参数 -g则为该组设置GID，默认GID为当前 最大GID+1
修改组|groupmod |参数-n修改名称，-g修改GID
删除组|groupdel| groupdel tgroup
添加用户|useradd | useradd test,参数 -g 添加至某组等， /etc/login.defs /etc/default/useradd 保存用户信息
设置密码|passwd| passwd test
修改系统已存在的组账号|usermod|
删除用户|userdel| 参数-r同时删除用户下目录

### 系统监控命令简述
#### top
> top命令显示了cpu的使用情况，每5秒刷新一次

信息  | 含义
---|---
PID|进程标识
USER|进程所属用户
PRI|进程的优先级
NI|nice级别
RSS|进程使用的物理内存
SHARE|该进程和其他进程共享内存的数据
STAT|进程的状态 S=休眠，R=运行，T=停止，D=中断休眠，Z=僵尸状态
%CPU|共享的CPU使用
%MEM|共享的物理内存
TIME|进程占用CPU时间
COMMAND|启动任务的命令行

#### iostat

> iostat 显示磁盘系统的使用情况，用来监控CPU利用率和磁盘利用率


信息  | 含义
---|---
%user | 用户级应用的CPU占用率
%nice | 加入nice优先级的用户级应用CUP占有率
%sys | system级的CPU占用率
%idle | 空闲的CPU

#### vmstat
>对进程、内存、页面I/O和CPU信息监控，可显示检测结果的平均值

参数 | 含义
---|---
vmstat 2 1| 第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数
r | 等待运行的进程数量
b | 阻塞的进程数量
swpd |虚拟内存已使用的大小，如果大于0，表示机器物理内存不足了
free | 空闲的物理内存的大小
buff  | Linux/Unix系统是用来存储，目录、权限等的缓存
cache| 文件缓冲，空闲的物理内存的一部分拿来做文件和目录的缓存
si  |每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了
so | 每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上
bi | 块设备每秒接收的块数量，即向一个块输出这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte
bo |块设备每秒发送的块数量，从一个块设备接收的块数量
in |每秒CPU的中断次数，包括时间中断
cs |每秒上下文切换次数
us |用户CPU时间
sy |系统CPU时间，如果太高，表示系统调用时间长
id |空闲 CPU时间，一般来说，id + us + sy = 100
wt |等待IO CPU时间。

#### free

>显示系统的所有内存的使用情况，包括空闲内存、被使用得内存和交换内存空间

- du 命令

命令 | 含义 
---|---
du -m | 以M显示文件夹下所有文件大小
du -k| 以K为单位
df -a|显示所有文件系统的磁盘使用情况
df -k|以K为单位
df -m|以m为单位
df -h|以易读的方式显示
df -t|列出文件类型

#### pmap

>显示一个或者多个进程使用内存的数量

信息 | 含义
---|---
Address| start address of map  映像起始地址
Kbytes|  size of map in kilobytes  映像大小
RSS| resident set size in kilobytes  驻留集大小
Dirty|  dirty pages (both shared and private) in kilobytes  脏页大小
Mode| permissions on map 映像权限: r=read, w=write, x=execute, s=shared, p=private (copy on write)
Offset|  offset into the file  文件偏移
Device|  device name (major:minor)  设备名

#### netstat -anlp
>查看端口占用情况

参数 | 含义
---|---
-a| (all)显示所有选项，默认不显示LISTEN相关
-t |(tcp)仅显示tcp相关选项
-u |(udp)仅显示udp相关选项
-n| 拒绝显示别名，能显示数字的全部转化成数字。
-l |仅列出有在 Listen (监听) 的服務状态
-p |显示建立相关链接的程序名
-r |显示路由信息，路由表
-e| 显示扩展信息，例如uid等
-s |按各个协议进行统计
-c| 每隔一个固定时间，执行该netstat命令。

### 进程命令 ps

参数 | 含义
---|---
-A| 列出所有的行程
-a |显示一个终端的所有进程
-x| 显示各个命令的具体路径
-p| pid 进程使用cpu时间
-u uid or username| 选择有效的用户
-g gid orgroupnam| 选择有效的用户组
U |username 显示用户下的所有进程，且显示各个明亮的详细路径
-f| 全部列出，配合使用：ps -fa 或 ps -fx ...
-l |长格式
-j|作业格式
v|以虚拟存储格式
s|以信号格式
-m|显示所有进程
-H|显示进程的层次,和其他命令合用，如ps-Ha
e|命令后显示环境,如 ps -d e
h|不显示第一行

其他常用：

参数 | 含义
---|---
ps -ef 或ps -aux| 查看进程
kill -9  |强制杀死进程
jobs| 查看中止或后台运行的进程
bg |把进程放在后台运行
fg |把进程放在前台运行
ctrl+c| 终止在前台运行的进程

### 远程操作命令

- shh
- scp

将 xx路径 下内容发送至 host主机 usr用户的 yy路径下
```
 scp -r  xx usr@host: yy
```
