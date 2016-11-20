---
title: Git 学习进阶篇-重写历史记录
date: 2016-11-12 17:58:27
description: "更深入介绍版本控制工具git的使用"
categories: [Git]
tags: [重写历史]
top: 18
---
## Git 重写历史记录

涉及命令：
```
#git commit -amend
#git rebase
#git reset 

```
## git commit --ament  
如果刚进行提交操作，发现提交信息写错了，或者还需要几个文件一起提交，那么使用--amend命令可以重新提交。

### 忘了添加某个文件，重新提交
``` 
$ git lol -- stage.txt                   //当前提交信息
* 3ee6f40 (HEAD -> master) efive commit~
* d4df151 five

$ touch stage2.txt
$ git add .                             //git rm 
$ git commit --amend
[master 6ba634c] two file to commit~
 Date: Thu Nov 17 23:39:39 2016 +0800
 2 files changed, 1 insertion(+)
 create mode 100644 stage2.txt

$ git lol
* 6ba634c (HEAD -> master) two file to commit~       //重新提交，覆盖了3ee6f40提交
* d4df151 five

```
### 忘了删除某个文件，重新提交


```
$ touch a.text b.txt                //添加两个文件
$ git add .
$ git commit -m 'test for git rm'

$ git lol                           //查看历史
* 2506ac0 (HEAD -> master) test for git rm
* 50c430c delete a file

$ git rm b.txt                      //这时，发现b.txt不应该提交，需要删除
rm 'b.txt'

$ git commit --amend                //重写历史，将暂存区内容作为提交的快照
[master bf8190a] test for git rm,and rm b.txt
 Date: Sun Nov 20 20:57:20 2016 +0800
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.text

$ git lol                           //查看历史，可以看到记录已被修改
* bf8190a (HEAD -> master) test for git rm,and rm b.txt
* 50c430c delete a file
* ea9691e add file remove.d

$ ls                                //b.txt文件也已经被删除
a.text  checkoutdir/  cleandir/  master.d  revertdir/  stage.txt

```

## git rebase
使用git commit --amend可以重写最近的一次提交，但是无法修改历史中更早的提交；git rebase则可以实现

### 修改多个提交说明
如果你想修改某次提交，需要参数 git rebase -i HEAD~n/SHA-1,如果是已经push到远程中心服务器的提交，需要慎重使用

- 文件准备
```
$ git lol -- a.txt
* 9bc0634 (HEAD -> master) fourth a.txt commit
* 1169361 third a.txt commit
* f868857 second a.txt commit
* e232dcd first a.txt commit

$ cat -- a.txt    //上面每次提交所对应的内容， git log的顺序是从 最新提交到历史提交
1111111       
2222222
3333333
4444444

```
- 执行命令 git rebase -i HEAD~ 修改前两次提交

>显示顺序 历史-最新，因为rebase是从历史开始修改,与git log 查看历史顺序相反

```
pick 1169361 third a.txt commit              
pick 9bc0634 fourth a.txt commit

# Rebase f868857..9bc0634 onto f868857 (2 command(s)) //修改的范围
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
```
- 使用edit，修改这两次提交信息
```
edit 1169361 third a.txt commitd
edit 9bc0634 fourth a.txt commit

//保存，退出
```
- 依次修改历史提交信息

```
$ git rebase -i HEAD~2
Stopped at 1169361b7859289b58ed36defcaa19d0d2e207b6... third a.txt commitd
You can amend the commit now, with

        git commit --amend         //告诉我们使用 --amend命令修改commit信息

Once you are satisfied with your changes, run

        git rebase --continue     //如果修改完毕，--continue继续下一步
A@Just-pc MINGW32 /g/git_test/git_repo (master|REBASE-i 1/2)  //看到rebase有两步
```

- 使用git commit --amend 来修改 SHA-1 为【1169361】的提交信息

```
$ git commit --amend
[detached HEAD ff5173e] after rebase:third a.txt commit  //修改后的提交信息
 Date: Sun Nov 20 21:11:11 2016 +0800
 1 file changed, 1 insertion(+)
 
$ git rebase --continue                         //下一步
Stopped at 9bc063423f76f8d4a33122943421651255356a69... fourth a.txt commit
...
@Just-pc MINGW32 /g/git_test/git_repo (master|REBASE-i 2/2)  

```
- 继续第二步，然后结束rebase
```
$ git commit --amend
[detached HEAD 89599bf] after rebase:fourth a.txt commit
 Date: Sun Nov 20 21:12:05 2016 +0800
 1 file changed, 1 insertion(+)

$ git rebase continue

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master|REBASE-i 2/2)
$ git rebase --continue
Successfully rebased and updated refs/heads/master.

$ git lol  //查看历史，近两次提交信息已修改完成
* 89599bf (HEAD -> master) after rebase:fourth a.txt commit
* ff5173e after rebase:third a.txt commit
* f868857 second a.txt commit
* e232dcd first a.txt commit

```
> 还有其他命令 git rebase (--continue | --abort | --skip) 亲自体验下咯

### 重排提交
可以直接删除某次历史提交
```
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git rebase -i HEAD~3
pick 1169361 third a.txt commit              
pick 9bc0634 fourth a.txt commit
pick 1169361 third a.txt commit  //直接删除
```
结果：

```
$ git rebase -i HEAD~3
Successfully rebased and updated refs/heads/master.

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git lol                         //已不存在 1169361
* ff5173e (HEAD -> master) after rebase:third a.txt commit
* f868857 second a.txt commit
* e232dcd first a.txt commit

```
> 使用rebase期间针对某一提交，也可以进行等操作，直到运行 git rebase --continue结束对这一提交的修改

### 线性分支

- 分支准备
```
$ git checkout -b rebase_branch      //从 ff5173e 引用检出
Switched to a new branch 'rebase_branch'

Administrator@Just-pc MINGW32 /g/git_test/git_repo (rebase_branch)
$ vim a.txt                                 //在分支rebase_branch修改、提交文件a.txt
$ cat a.txt
1111111
2222222
3333333
initial on rebase_marge                     //添加的内容
                           
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ cat a.txt                                 //切换到master分支上，修改文件a.txt
1111111
2222222
3333333
modify on master 
Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git lol                                  //查看历史，从ff5173e出现tree的节点
* 95486b0 (HEAD -> master) modify a.txt
| * a59e983 (rebase_branch) rebase_branch innitail commit
|/
* ff5173e after rebase:third a.txt commit
* f868857 second a.txt commit

```
目的：
>不使用marge命令，来使得rebase_branch分支相当于是从 master的最新引用95486b0上checkout出来的，相当于是线性分支

- 线性融合

```
$ git rebase master
...
CONFLICT (content): Merge conflict in a.txt
...
When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
```
>提示中，告诉我们是在处理两个分支有内容冲突的文件 a.txt,使用 --continue继续当前操作，或者--skip 跳过，或者 --abort放弃当前操作

我们这里选择继续当前操作：

```
$ vim a.txt              //修改冲突文件内容
1111111
2222222
3333333
<<<<<<< 95486b0adc5c1e743ddcb1df8e7eaff5aeb97670
modify on master
=======
initial on rebase_marge
>>>>>>> rebase_branch innitail commit

$ git add .             //然后，add进暂存区

Administrator@Just-pc MINGW32 /g/git_test/git_repo (rebase_branch|REBASE 1/1)
$ git rebase --continue //不使用commit，直接 --continue
Applying: rebase_branch innitail commit

Administrator@Just-pc MINGW32 /g/git_test/git_repo (rebase_branch)
$ git lol
* dcc2a9a (HEAD -> rebase_branch) rebase_branch innitail commit  //已成为线性得了

```

## git reset

当上面的 rebase_branch分支执行过线性合并后，其实在历史记录中，使用git log已经找不到之前的修改了，但是如果我们需要还原rebase操作又该怎么呢？

方法: git reflog与git reset组合

 命令|说明
---|---
git reflog |可以查看对master分支的所有引用修改
git reset --hard | 暂存区和工作区都被某一历史还原 
git reset --mixed | 暂存区被还原 (默认)
git reset --soft | 不还原

>相同之处：git reset (--hard | --mixed | --soft) 都将分支和HEAD引用，指向reset的commit

- 查看所有引用master的历史提交
```
$ git reflog    //a59e983 HEAD@{6} 是第一次在分支rebase_branch 上的提交
dcc2a9a HEAD@{0}: rebase finished: returning to refs/heads/rebase_branch
dcc2a9a HEAD@{1}: rebase: rebase_branch innitail commit
95486b0 HEAD@{2}: rebase: checkout master
a59e983 HEAD@{3}: checkout: moving from master to rebase_branch
95486b0 HEAD@{4}: commit: modify a.txt
ff5173e HEAD@{5}: checkout: moving from rebase_branch to master
a59e983 HEAD@{6}: commit: rebase_branch innitail commit
ff5173e HEAD@{7}: checkout: moving from master to rebase_branch

$ git reset --hard HEAD@{6}
HEAD is now at a59e983 rebase_branch innitail commit

$ git lol
* 95486b0 (master) modify a.txt
| * a59e983 (HEAD -> rebase_branch) rebase_branch innitail commit
|/
* ff5173e after rebase:third a.txt commit

```

### END
- git commit --amend使用新的提交覆盖当前提交
- git rebase比git commit --amend可修改的历史范围更加广泛，还可以处理分支操作，
- git reset依据参数还原某些历史

注：对于已经push到远程服务中心，即已经共享的提交，需要谨慎操作！~

