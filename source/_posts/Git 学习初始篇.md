---
title: Git 学习初始篇
date: 2016-11-12 17:58:27
description: "介绍版本控制工具git的使用"
categories: [Git]
tags: []
top: 15
---
# Git
## what is Git

重点：强大的代码管理工具！

### 为什么使用？

重点：使用github社区必备，而且确实方便高效。

## git与svn区别
- svn将代码放于中央服务器；git属于分布式版本控制系统，每个版本库都可以是独立的
>分布式的git，可以完成很多离线操作，svn则做不到，如果与中央服务器断开，则什么都做不了
- svn保存的是当前版本与上一版本的差异,即只保存了从 A到A.1 改动的部分，而GIT保存的是完整的文件

![image](http://og6e19zce.bkt.clouddn.com/git_learn1.png)

- git可以实现分支合并操作
- git效率更高
- git对版本修改更加灵活，版本之间通过命令即可实现代码切换

## git的简单配置

假设已安装好git

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
### 基本使用

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
如果需要忽略某些文件，而不需要被跟踪？可以在需要的目录位置创建 **.gitignore** 文件

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


