---
title: Git 学习进阶篇-分支与合并
date: 2016-11-12 17:58:27
description: "更深入介绍版本控制工具git的使用"
categories: [Git]
tags: [分支，合并]
top: 16
---
## Git brach 分支
Git可以创建多个分支，用于对不同版本代码分别进行维护操作。

>Git对每个分支**默认读取最新commit索引**。

branch操作常用命令：

命令 | 描述| 实例
---|---|---
git branch xx | 创建分支| git branch test,创建test分支
git checkout xx| 切换分支| git checkout test，切换到分支test


```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git branch test               //创建test分支
$ ls   //查看master分支下文件，只有master.d
master.d
$ git checkout test             //切换到 test分支
Switched to branch 'test'
```

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (test)
$ vim test.d                    //添加文件 test.d
$ git add .
$ git commit -m 'first commit on branch test'
[test be2635d] first commit on branch test
 1 file changed, 1 insertion(+)
 create mode 100644 test.d

```

```
$ git checkout master           //切换为maste分支
Switched to branch 'master'

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ ls                            //maser分支显示的是之前的最新的commit提交引用
master.d
```

>如何让引用指向某一固定提交呢？如指向某些发布的版本， tag!

## Git tag

分类：
- 轻量级，本地的引用；
- 带注解的，就是之前存储在g**it对象中的Git tag对象**。

相关命令：

命令 | 描述  | 实例
---|---|---
git tag | 创建本地tag| git tag "V0" SHI-ID
git tag -a | 创建注解tag
git show | 查看tag内容| git show V0
git checkout | 切换到tag| git checkout V0

### 首先，查看当前的历史记录
```
$ git config --global alias.lol  "log --oneline --all --decorate --graph"  //定义别名
$ git lol   // 显示所有提交信息

* be2635d (test) first commit on branch test
* 629aa7f (HEAD -> master) second commit on master
* 5154992 master.d first time commit

```
### create tag

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git tag "V0" 5154992              //本地
$ git tag -a "MASTER_FIRST" 5154992 //注解

```
再次查看历史

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git lol                           // 在索引 5154992中以包含创建的两个tag
* be2635d (test) first commit on branch test
* 629aa7f (HEAD -> master) second commit on master
* 5154992 (tag: V0, tag: MASTER_FIRST) master.d first time commit
```
查看创建的tag内容

>其中，看两者不同之处,注解方式的**MASTER_FIRST** tag,是一个tag对象


```
$ git show V0
commit 51549925f380bd46c291f02d8988d368c28ecaab
Author: xx
Date:   Sun Nov 13 13:01:12 2016 +0800

```

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git show MASTER_FIRST
tag MASTER_FIRST
Tagger: xx
Date:   Sun Nov 13 13:30:00 2016 +0800
MASTER_COMMIT
```
### checkout tag

>在创建tag时，已经可以看到**tag也是指向某一个commit引用**，所以可以使用checkout命令，切换到tag

checkout tag V0:

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git checkout V0
Note: checking out 'V0'.

You are in 'detached HEAD' state. You can look around, make experimental
...
you may do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 5154992... master.d first time commit

```

>根据提示信息，此时处于 'detached HEAD' 状态，表示当前HEAD指向commit，而不是某个分支;我们在 **'detached HEAD'状态下的操作不会被保存**，除非我们使用 'git checkout -b <new-branch-name>' 命令，依据当前HEAD索引，创建新的分支，并切换到新的分支上。

为了保存操作信息，创建按照提示，创建新的分支 branch_V0
```
Administrator@Just-pc MINGW32 /g/git_test/git_repo ((MASTER_FIRST))
$ git checkout -b branch_V0
Switched to a new branch 'branch_V0'

```

对branch_V0分支进行操作，**如果对数据进行了修改，则必须进行 commit或者 stash 否则无法checkout其他分支**
```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_V0)
$ ls                                  //查看文件
master.d 
$ vim master.d                        //修改master.d文件
$ git add .                           //尝试添加到暂存区，发现遇到error

Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_V0)
$ git checkout master
error: Your local changes to the following files would be overwritten by checkout:
        master.d
Please, commit your changes or stash them before you can switch branches.
```
### git stash

可以想到，在开发过程中，当前修改还没有完成，又不得不去操作其他分支，如master分支，这时候使用stash比较合适

相关命令：

命令|描述
---|---
git stash | 切换分支前，保存没有提交的修改
git stash save -a |保存当前操作记录
git stash list| 查看保存的stash 记录 
git stash pop | 还原所有stash保存时的操作(如之前我们操作matser.d文件，还未提交)
git stash pop --index stash@{n} | 还原某个stash
git stash apply --index stash@{n} | 还原某个stash 但不会删除 stash
git stash drop stash@{n} | 删除某个stash，如果不加 stash则删除所有


>**git stash pop 与 git stash apply 区别在于前者会删除stash ，而 apply则不会删除 stash**


```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_V0)
$ git stash save -a 'stash_on_bracnV0' //保存 stash
Saved working directory and index state On branch_V0: stash_on_bracnV0
HEAD is now at 5154992 master.d first time commit
$ ls  //上面提示信息，表示已恢复到为修改之前，即tag V0所指向索引的内容
master.d

Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_V0)
$ cat master.d //查看内容，这里看到和 master第一次提交内容相同
111111111

```
当完成master分支后，返回branch_V0分支，继续之前的修改，这时候需要查看stash中都有哪些内容。

```
$ git stash list  //看到stash@{0} 引用指向我们创建的 stash_on_branchV0
stash@{0}: On branch_V0: stash_on_bracnV0

```
还原 stash保存的内容，**执行pop操作后，会同步删除还原的stash**
```
$ git stash pop --index stash@{0}  //还原stash
$ cat master.d
111111111                          //master分支保存的索引内容

modify on branch_V0                //这里是在brachV0上保存的索引内容

$ git stash list                   // 查看stash 发现 stash_on_branchV0已经不存在了
Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_V0)
$

```

验证git stash apply操作，重新保存一次 stash
```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_V0)
$  git stash save -a 'stash_second_on_branchV0' //保存stash
$ cat master.d                        // 查看文件内容
111111111 
$ git stash list                      // 还原前查看stash list
stash@{0}: On branch_V0: stash_onsecond_on_branchV0

$ git stash apply --index stash@{0}   //还原 stash
On branch branch_V0
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   master.d

$ cat master.d                       //查看文件内容
111111111
modify on branch_V0
$git stash list                      //apply后查看stash仍然存在    
stash@{0}: On branch_V0: stash_onsecond_on_branchV0


```

## Git merge 合并
>Git merge命令用来合并分支上的操作

相关命令:
命令 | 描述 | 示例
---|---|---
git merge | 当前分支合并某一分支中文件 | git merge test(分支名)
git merge --abord | 放弃合并操作| git merge --abord test


分支准备：
- master分支
```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ cat master.d                  
111111111
222222222

$ git checkout -b branch_merge //创建分支 branch_merge，（是master最新索引）
Switched to a new branch 'branch_merge'
```
- branch_merge分支
```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_merge)
$ vim master.d 
$ git add .
$ git commit -m 'commit on branch_merge' master.d
$ cat master.d  //branch_merge 分支内容
111111111
222222222
modify on branch branch_marge
```
- branch_V0 分支
```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (branch_merge)
$ git checkout branch_V0  
$ cat master.d   // branch_V0 分支内容
111111111
modify on branch_V0
```

现将 branch_V0分支 和 branch_merge分支内容合并到 master上
>区别：barch_V0分支对master.d文件的修改，是基于某一索引，从之前可以知道，是基于mastr的第一次索引；而 branch_merge分支是基于master的最新索引，也就相当于在master基础上修改

先查看下log

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git lol
* 872942d (branch_merge) commit on branch_merge
| * 3b9ceaa (branch_V0) commit on branch_V0
| | * be2635d (test) first commit on branch test
| |/
|/|
* | 629aa7f (HEAD -> master) second commit on master
|/
* 5154992 (tag: V0, tag: MASTER_FIRST) master.d first time commit

```
### 合并branch_merge

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git merge branch_merge
Updating 629aa7f..872942d
Fast-forward
 master.d | 1 +
 1 file changed, 1 insertion(+)
 
$ git status                    //很干净，没有需要提交的文件
On branch master
nothing to commit, working directory clean
```
从上面可以看到状态 'Fast-forward' 表示不需要进行提交等操作，和之前说的分支区别表示的意义相同。

查看master.d内容和日志

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ cat master.d
111111111
222222222
modify on branch branch_marge   //分支内容已合并到master上

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git lol
* 872942d (HEAD -> master, branch_merge) commit on branch_merge
| * 3b9ceaa (branch_V0) commit on branch_V0
| | * be2635d (test) first commit on branch test
| |/
|/|
* | 629aa7f second commit on master
|/
* 5154992 (tag: V0, tag: MASTER_FIRST) master.d first time commit

```
### 合并 branch_V0 

```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git merge branch_V0
Auto-merging master.d
CONFLICT (content): Merge conflict in master.d
Automatic merge failed; fix conflicts and then commit the result.
```
提示状态：Auto-mergign failed，需要修复冲突，然后提交，于是需要编辑 master.d文件


```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master|MERGING)
$ vim master.d

//以下是master.d文件中的内容

111111111
<<<<<<< HEAD
222222222
modify on branch branch_marge
=======

modify on branch_V0
>>>>>>> branch_V0
~

```
> 其中，"<<<<<<< HEAD" 与 "=======" 之间表示mastet上分支内容； "=======" 与 ">>>>>>> branch_V0"之间表示branch_V0分支上内容；根据需要进行修改(删除分支标识)后，将文件添加进暂存区,然后提交。


```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master|MERGING)
$ git add master.d

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master|MERGING)
$ git commit -m 'merge branch_V0' master.d  
fatal: cannot do a partial commit during a merge.

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master|MERGING)
$ git commit  //在 MERGIGN状态下，提交直接使用git commit
[master d8e6e7d] Merge branch 'branch_V0'

```

## END
- Git对分支的操作，会在分支上产生不同的引用;
- checkout切换分支，则是将HEAD切换到不用的引用上。
- tag可以指定某一索引的引用
- stash可以保存在某一分支上未完成的修改，实际上是对工作区、暂存区内容的还原
- marge对分支内容进行合并。