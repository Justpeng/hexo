---
title: Git 学习初始篇
date: 2016-11-12 17:58:27
description: "更深入介绍版本控制工具git的使用"
categories: [Git]
tags: [版本控制]
top: 11
---
# Git
## what is Git

重点：强大的分布式、高效代码管理工具！

### 为什么使用？

重点：使用github社区必备，而且确实方便高效。

## 版本控制
**Subversion ** : 集中化的版本控制系统,有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新

![image](http://ogrlfqipo.bkt.clouddn.com/svn_1.png)

- 优点：每个人都可以在一定程度上看到项目中的其他人正在做些什么。 而管理员也可以轻松掌控每个开发者的权限，并且管理一个 CVCS 要远比在各个客户端上维护本地数据库来得轻松容易。

- 缺点：是中央服务器的单点故障。发生故障谁都无法提交更新，也就无法协同工作。

**Git**:客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。 

![image](http://ogrlfqipo.bkt.clouddn.com/git_1.png)

## git与svn区别

主要区别：对数据的处理方法
- Git直接记录完整文件，SVN则是记录基本文件和每个文件随时间逐步累积的差异

![image](http://ogrlfqipo.bkt.clouddn.com/git_2.png)

![image](http://ogrlfqipo.bkt.clouddn.com/svn_2.png)
- Git 所有操作几乎都是本地执行，不用连网，处理速度飞快。
> 在没有网络的情况也可以访问历史，进行本地修改、提交等操作，只需要在有网络时push到远程仓库
- Git 时刻保持数据完整性 
>在commit前，git会对数据进校验和（checksum）计算，并将此结果作为数据的唯一标识和索引。检查数据是否修改、缺失。

> Git 使用 SHA-1 算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个 SHA-1 哈希值，作为指纹字符串。该字串由 40 个十六进制字符（0-9 及 a-f）组成，看起来就像是：

```
24b9da6552252987aa493b52f8696cd6d3b00373
```

Git 的工作完全依赖于这类指纹字串，所以你会经常看到这样的哈希值。实际上，所有保存在 Git 数据库中的东西都是用此哈希值来作索引的，而不是靠文件名。 



## git的简单配置

假设你已安装好git

### git操作级别

优先级 (高到低) |级别 (低到高) | 描述
---|---|--
1| local | 本地级别
2| global|用户级别
3| system | 系统级别

### git文档查看

方式 | 示例
---|---
1| git config --help
2| git help config


### 命令技巧
- 输入 git conf 双击 tab 键 可自动补充内容，类似于 linux中自动补充
- 输入 git config -- ，双击 tab 可显示可用参数

```
$ git config --
--add              --global           --rename-section
--file=            --list             --replace-all
--get              --local            --system
--get-all          --name-only        --unset
--get-regexp       --remove-section   --unset-all

Administrator@Just-pc MINGW32 ~/Desktop
$ git config --

```
### 使用前配置

- 设置用户和邮箱:相当于身份证

```
git config --global user.name 'xx'
git config --global user.email 'xx@yy'
```
- 1. 查看用户
```
git config --get user.name  //当前使用的,默认最新的

git config --list --global  //用户所有user.name

```
- 2. 删除用户

```
git config --global --unset user.name 用户名
```

- config命令
如 创建别名
```
git config --global alias.ci commit
```
查看效果，多了个ci，也就是commit命令别名

```
$ git c
checkout      cherry-pick   citool        clone         config
cherry        ci            clean         commit


```

```
$ git config --global alias.lol 'log --oneline'
$ git lol
91b757f Merge remote-tracking branch 'origin/master'
f352b6e 添加博客
2c79fcb Create README.md
21155b9 Initial commit
```
## Git的存储
Git存储只关系文件内容

Git使用40个16进制字符的SHA-1 Hash来存储唯一标识对象

Git有四种对象：
- blob 文本文件、二进制文件、链接文件
- tree 目录
- commit 历史提交
- tag 指向固定的历史提交

四种对象之间关系：一个tag指向一个commit,commit指向一个tree对象，tree对象中也可包含其他tree对象和blob对象；若有像个相同内容的文件，则指向在相同blob，而文件名等信息存储在 tree对象中

![image](http://og6e19zce.bkt.clouddn.com/git_learn2.png)

Git将对象存储在仓库中

## Git仓库
- 创建Git仓库: 
```
git init  //新建
git clone  //克隆
```
创建指定仓库
```
git init git_repo //创建时 git_repo下有.git目录
git init --bare git_bare_repo  //git_bare_repo下没有.git目录，不包含工作区
git init  //直接在当前目录下新建

```
克隆已有仓库

```
git clone git_repo git_clone  //1 本地仓库克隆，指定路径

git clone https://github.com/xxx  //2 克隆github远程仓库到当前目录

```
## Git工作流程

Git仓库有三个区域：
- 工作区working direcoty;暂存区staging area,历史仓库history respository

![image](http://og6e19zce.bkt.clouddn.com/git_learn3.png)

- 区域间文件传递：

![image](http://og6e19zce.bkt.clouddn.com/git_learn4.png)
涉及的命令：

```
git add  //将工作区文件添加到暂存区
git add -A //将工作区所有文件添加早暂存区
git commit //将暂存区添加到历史区
git status //查看工作区与暂存区文件区别
git rm  //删除暂存区文件
git mv //移动/重命名 暂存区文件，一系列操作的组合(先删除后添加)
gitingnore  //忽略工作区中某些不希望被加入暂存区和历史区的文件
```
而，如果只是 mv 命令(没有git)，则操作的只是本地文件

For Example：

```
Administrator@Just-pc MINGW32 /g/test/git_repo (master)
$ touch test.txt a.java  //创建文件
$ git add test.txt a.java  //添加至缓存
$ git status  //查看文件状态

On branch master
Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   a.java
        new file:   test.txt


Administrator@Just-pc MINGW32 /g/test/git_repo (master)
$ git commit -m "add files first time"  //提交至历史区

[master (root-commit) 02463dd] add files first time
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.java
 create mode 100644 test.txt

```

这里说下**Git中文件的状态** ：未跟踪和已跟踪，通过**git status**命令查看

>请记住，工作目录下面的所有文件都不外乎这两种状态：已跟踪或未跟踪。已跟踪的文件是指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，它们的状态可能是未更新，已修改或者已放入暂存区。而所有其他文件都属于未跟踪文件。它们既没有上次更新时的快照，也不在当前的暂存区域。初次克隆某个仓库时，工作目录中的所有文件都属于已跟踪文件，且状态为未修改。

简单理解的话，只要是已经**被Git命令操作过的，纳入Git管理**的文件都是已跟踪文件，没有被Git命令操作的文件都是未跟踪。比如在工作区touch新文件，则状态是未跟踪，使用git add 添加进缓存区是，就是已跟踪了。

其中，Git中文件状态变化周期如下图：

![image](http://ogrlfqipo.bkt.clouddn.com/git_3_filestatus.png)








如果需要**忽略某些文件**，而不需要被跟踪？可以在需要的目录位置创建 **.gitignore** 文件

通配符匹配规则：
匹配符 | 规则 | 举例
---|---|---
a.txt | 过滤具体的 a.txt文件 |
*~ |中间类型文件| 如vim~ 匹配vim操作产生的文件
* | 匹配多个字符| *.txt 所有txt文件都被忽略
[]| 包含单个字符的匹配列表| *.[ab] 过滤以a或b结尾的文件
/ | 匹配目录| /demo 过滤当前目录demo文件夹及文件夹下所有文件
**/| 匹配任何目录下的某文件夹|  两个星号/demo，过滤任何目录(以.gitignore文件所在目录为根目录）下包含demo文件夹及其文件夹下的文件
! | 不过滤 | !a.txt 不过滤a.txt文件，若文件本身以 !开始，则需要加"\\" 转义，如"\\!a.txt" 不过滤 !a.txt文件

For Example:
```
Administrator@Just-pc MINGW32 /g/test/git_repo (master)
$ vim .gitignore  // 添加规则 *.css，忽略.css文件
$ touch table.css

Administrator@Just-pc MINGW32 /g/test/git_repo (master)
$ git status   //已忽略 table.css
On branch master
nothing to commit, working directory clean

```


