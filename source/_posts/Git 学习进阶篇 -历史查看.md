---
title: Git 学习进阶篇-历史查看
date: 2016-11-13 23:58:27
description: "学习如何查看git的历史"
categories: [Git]
tags: [git历史]
top: 16
---
# Git历史记录的查看与对比
主要涉及到三个命令：git log 、git diff 、git show

## git log 命令使用
在项目提交了若干记录，==git log== 命令可以查看提交历史：

```
$ git log
commit d8e6e7d0a2bbc4483bf5c0d884daec7bfc37b861      
Merge: 872942d 3b9ceaa                              //合并分支的两个 SHA-1校验  
Author: just <just@163.com>
Date:   Sun Nov 13 15:29:44 2016 +0800

    Merge branch 'branch_V0'

commit 872942db74378b9be73d3fc22a44350ddd039277     //SHA-1校验和
Author: just <just@163.com>                         //作者名字和邮件
Date:   Sun Nov 13 14:52:09 2016 +0800              //提交时间

    commit on branch_merge                          //提交说明

commit 3b9ceaa45c8f5e3ab2521b5f549a58901c1dec60
Author: just <just@163.com>
Date:   Sun Nov 13 14:47:49 2016 +0800

    commit on branch_V0

commit 629aa7f0e58853c39ae22e1b6ab06a5d148eac13
Author: just <just@163.com>
Date:   Sun Nov 13 13:03:03 2016 +0800

    second commit on master

commit 51549925f380bd46c291f02d8988d368c28ecaab
Author: just <just@163.com>
Date:   Sun Nov 13 13:01:12 2016 +0800

    master.d first time commit

```
>默认不用任何参数的话，==git log== 会按提交时间列出所有的更新，最近的更新排在最上面。每次更新都有一个 SHA-1 校验和、作者的名字和电子邮件地址、提交时间，最后缩进一个段落显示提交说明。

在使用git log命令查看历史记录时，使用 空格、enter键、B、以及上下键来进行翻页，使用q退出历史查看

### 具体使用

常用命令：

命令 | 描述 | 作用
---|---|---
git log -p | 展开显示每次提交的内容差异 | 清晰对比每次commit的变化情况
git log --stat | 仅显示简要的增改行数统计| 列出修改过的文件，以及其中添加和移除的行数，并在最后列出所有增减行数小计
git log --oneline|　将日志信息简要为一行显示

1. git log -p -2

```
$ git log -p
commit 257f3a287fd04f1bbf9328e9b80398353569e6fa
Author: xx
Date:   Mon Nov 14 22:29:37 2016 +0800

    modify master.d

diff --git a/master.d b/master.d
index ded902a..61ed066 100644
--- a/master.d
+++ b/master.d
@@ -1,4 +1,7 @@
 111111111
 222222222
 modify on branch branch_marge
-modify on branch_V0               //显示出内容差异，添加和删除
+modify on branch
+
+
+

commit d8e6e7d0a2bbc4483bf5c0d884daec7bfc37b861
Merge: 872942d 3b9ceaa
Author: xx
Date:   Sun Nov 13 15:29:44 2016 +0800

    Merge branch 'branch_V0'

```
2. git log --stat

```
$ git log --stat
commit 257f3a287fd04f1bbf9328e9b80398353569e6fa
Author: lipeng <lipeng_ly@163.com>
Date:   Mon Nov 14 22:29:37 2016 +0800

    modify master.d

 master.d | 5 ++++-    
 1 file changed, 4 insertions(+), 1 deletion(-)   //显示更新的统计信息

```
3. git log --oneline

```
$ git log --oneline
257f3a2 modify master.d             //简要为一行显示
d8e6e7d Merge branch 'branch_V0'
872942d commit on branch_merge
3b9ceaa commit on branch_V0
629aa7f second commit on master
5154992 master.d first time commit

```
其他命令：

命令 | 描述
---|---
-p	|按补丁格式显示每个更新之间的差异。
--word-diff|	按 word diff 格式显示差异。
--stat	|显示每次更新的文件修改统计信息。
--shortstat	|只显示 --stat 中最后的行数修改添加移除统计。
--name-only	|仅在提交信息后显示已修改的文件清单。
--name-status	|显示新增、修改、删除的文件清单。
--abbrev-commit	|仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
--relative-date|	使用较短的相对时间显示（比如，“2 weeks ago”）。
--graph|	显示 ASCII 图形表示的分支合并历史。
--pretty|	使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。
--oneline	|--pretty=oneline --abbrev-commit 的简化用法。

命令--word-diff显示的是行层面的对比，将其与 -p 组合可以获得单词层面的对比
```
$ git log -p --word-diff
commit 257f3a287fd04f1bbf9328e9b80398353569e6fa
Author: xx
Date:   Mon Nov 14 22:29:37 2016 +0800

    modify master.d

diff --git a/master.d b/master.d
index ded902a..61ed066 100644
--- a/master.d
+++ b/master.d
@@ -1,4 +1,7 @@
111111111
222222222
modify on branch branch_marge
modify on [-branch_V0-]{+branch+}   //两次提交涉及行层面的 单词对比

```
新增加的单词被 {+ +} 括起来，被删除的单词被 [- -] 括起来

此外，==--pretty==可以使用完全不同于默认格式的方式展示提交历史，如：
```
$ git log --pretty=oneline      //与git log --oneline显示不同
257f3a287fd04f1bbf9328e9b80398353569e6fa modify master.d
d8e6e7d0a2bbc4483bf5c0d884daec7bfc37b861 Merge branch 'branch_V0'
872942db74378b9be73d3fc22a44350ddd039277 commit on branch_merge

```
==format==，可以定制要显示的记录格式，这样的输出便于后期编程提取分析，如
```
$ git log --pretty=format:"%h - %an, %ar : %s"
257f3a2 - just, 29 minutes ago : modify master.d
d8e6e7d - just, 31 hours ago : Merge branch 'branch_V0'
872942d - just, 32 hours ago : commit on branch_merge
3b9ceaa - just, 32 hours ago : commit on branch_V0
629aa7f - just, 34 hours ago : second commit on master
5154992 - just, 34 hours ago : master.d first time commit

```
其参数及含义：

命令 | 说明
---|---
%H	|提交对象（commit）的完整哈希字串
%h	|提交对象的简短哈希字串
%T	|树对象（tree）的完整哈希字串
%t	|树对象的简短哈希字串
%P	|父对象（parent）的完整哈希字串
%p	|父对象的简短哈希字串
%an	|作者（author）的名字
%ae	|作者的电子邮件地址
%ad	|作者修订日期（可以用 -date= 选项定制格式）
%ar	|作者修订日期，按多久以前的方式显示
%cn	|提交者(committer)的名字
%ce	|提交者的电子邮件地址
%cd	|提交日期
%cr	|提交日期，按多久以前的方式显示
%s	|提交说明

用 oneline 结合 --graph 选项(或 format 时)，可以看到开头多出一些 ASCII 字符串表示的简单图形，形象地展示了每个提交所在的分支及其分化衍合情况。

```
$ git log --graph --oneline
* 257f3a2 modify master.d
*   d8e6e7d Merge branch 'branch_V0'
|\
| * 3b9ceaa commit on branch_V0
* | 872942d commit on branch_merge
* | 629aa7f second commit on master
|/
* 5154992 master.d first time commit

```
### 限制输出长度
除了定制输出格式的选项之外，git log 还有许多非常实用的限制输出长度的选项，也就是只输出部分提交信息。

命令|说明
---|---
-(n)	|仅显示最近的 n 条提交
--since, --after	|仅显示指定时间之后的提交。
--until, --before|	仅显示指定时间之前的提交。
--author	|仅显示指定作者相关的提交。
--committer	|仅显示指定提交者相关的提交。

下面的命令列出所有最近两周内的提交：
$ git log --since=2.weeks

```
$ git log --since=2.weeks
commit 257f3a287fd04f1bbf9328e9b80398353569e6fa
Author:xx
Date:   Mon Nov 14 22:29:37 2016 +0800

    modify master.d

```
十小时前：
```
$ git log --since=10.hour
commit 257f3a287fd04f1bbf9328e9b80398353569e6fa
Author: xx
Date:   Mon Nov 14 22:29:37 2016 +0800

    modify master.d

```

另一个真正实用的git log选项是路径(path)，如果只关心某些文件或者目录的历史提交，可以在 git log 选项的最后指定它们的路径，使用 -- 与之前命令分开。

在项目中有文件 dir1/test.d

```
$ git log --pretty=oneline -- dir1/test.d
cc5ebf86a2e294856c4d79c6e0d2323d46cf8980 111111111


```
### 图形化工具查看
==gitk== 命令

## git diff 
命令 | 描述
---|---
git diff |输出工作区与暂存区的差异
git diff --cached | 输出暂存区与历史区差异
git diff | --cached 指暂存区中的差异
git diff HEAD~2| 某次提交的差异
git diff HEAD~3 HEAD~2 | 某两次提交对象之间的差异
git diff |--color-words 单词间差异

1. 本地工作区与暂存区
```
$ vim master.d  //先对工作区的mater.d 文件进行修改
$ git diff      //查看差异
diff --git a/master.d b/master.d
index 2c9fe9f..d76916d 100644
--- a/master.d
+++ b/master.d
@@ -3,5 +3,5 @@
 modify on branch branch_marge
 modify on branch

-efe
+ef223e

```

2. 暂存区与历史区


```
$ git add .
$ git diff --cached
diff --git a/master.d b/master.d
index 2c9fe9f..d76916d 100644
--- a/master.d
+++ b/master.d
@@ -3,5 +3,5 @@
 modify on branch branch_marge
 modify on branch

-efe
+ef223e

```
3. 查看历史提交差异

- **HEAD** 永远指向最新的提交的对象，HEAD^^ 近第二次次提交的对象，或者使用 HEAD~n n指正整数

```
$ git diff HEAD~2 -- master.d
diff --git a/master.d b/master.d
index ded902a..d76916d 100644
--- a/master.d
+++ b/master.d
@@ -1,4 +1,7 @@
 111111111
 222222222
 modify on branch branch_marge
-modify on branch_V0
+modify on branch
+
+ef223e
+

```
- git diff --cached 指暂存区中的差异

```
$ git diff  --cached HEAD~2 -- master.d
diff --git a/master.d b/master.d
index ded902a..d76916d 100644
--- a/master.d
+++ b/master.d
@@ -1,4 +1,7 @@
 111111111
 222222222
 modify on branch branch_marge
-modify on branch_V0
+modify on branch
+
+ef223e
+

```
- git diff HEAD~3 HEAD~2  某两次提交对象之间的差异

```
$ git diff HEAD~3 HEAD~2 -- master.d
diff --git a/master.d b/master.d
index 1800c80..ded902a 100644
--- a/master.d
+++ b/master.d
@@ -1,3 +1,4 @@
 111111111
 222222222
 modify on branch branch_marge
+modify on branch_V0
```
- git diff --color-words 单词间差异
```
$ git diff --color-words
diff --git a/master.d b/master.d
index d76916d..e7bd13e 100644
--- a/master.d
+++ b/master.d
@@ -3,5 +3,5 @@
modify on branch branch_marge
modify on branch

ef223eef223eabc

```
## git show
1. ==git show== 命令接受git对象作为参数,输出完整提交的差异信息，如：


命令 | 描述
---|---
 git show SHA-1| 输出某一提交对象的记录
git show master |输出示master分支最新提交引用
git show master~2 | 输出master分支上近第二次提交记录
git show --oneline(其他)|格式化输出

```
$ git lol               //查看历史
* cc5ebf8 (HEAD -> master) 111111111
* 257f3a2 modify master.d
*   d8e6e7d Merge branch 'branch_V0'

$ git show cc5ebf8
commit cc5ebf86a2e294856c4d79c6e0d2323d46cf8980
...                //显示详细差异信息
 modify on branch branch_marge
 modify on branch
+efe

```
>git show **cc5ebf8** ，显示SHA-1校验和为cc5ebf8的提交记录差异，当然这里 与命令 git how 和 git show master是等价的，因为都是显示 matser分支上最新的提交。


```
$ git show --oneline master~2  //格式化输出
diff --cc master.d      //将记录信息缩短为一样，下面都是差异
index 1800c80,eb4949f..ded902a
--- a/master.d
+++ b/master.d
@@@ -1,3 -1,3 +1,4 @@@
  111111111
 -
 +222222222
 +modify on branch branch_marge
+ modify on branch_V0
```
2. 查看其他git对象

**tag**对象
```
$ git tag
MASTER_FIRST
V0

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git show V0
commit 51549925f380bd46c291f02d8988d368c28ecaab
....
@@ -0,0 +1 @@
+111111111

```
**tree**对象

```
$ git show --format=%t master   //简要输出tree对象：format %t 
0b3d703                         //tree对象 SHA-1

diff --git a/dir1/test.d b/dir1/test.d
new file mode 100644
index 0000000..e69de29

$ git show 0b3d703             //输出tree差异
tree 0b3d703

dir1/                         //tree对象中的文件
master.d


```
**blob**对象


```
$ git show --format=%t master~2
389039c                          //tree对象

diff --cc master.d      
index 1800c80,eb4949f..ded902a   //blob对象 

$ git show 1800c80
111111111
222222222
modify on branch branch_marge
```


## END.
- git show 输出以git对象为参数的提交差异信息
- git log则可以更加详细的差异统计信息
- git diff 可以输出更加具体的提交之间的差异，还可以比较 **不同区** 之间的差异





















