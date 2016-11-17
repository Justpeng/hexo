## Git的撤销操作
这里介绍一些基本的撤消操作相关的命令。当然，有些撤销操作是不可逆的

涉及命令：

命令 | 说明
---|---
git checkout |还原工作区内容，即暂存区内容覆盖工作区内容,
git reset |还原暂存区内容，即撤销 git add操作
git clean|删除工作区中没有被git跟踪的文件
git revert|删除一些已提交的历史文件，当然会用新的覆盖旧的

## git checkout

git chekcout 既有切换分支功能，也可以还原工作区文件

```
$ git checkout --help
#NAME
git-checkout - Switch branches or restore working tree files
```

- 以当前最新历史记录还原：

```
$ vim master.d               //修改master.d文件
$ git diff                   //本地与暂存区差异
diff --git a/master.d b/master.d
index e7bd13e..3077768 100644
--- a/master.d
+++ b/master.d
@@ -4,4 +4,6 @@ modify on branch branch_marge
 modify on branch

 ef223eabc
+test gitcheckout
+

$ git checkout -- master.d      //使用git checkout撤销修改

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git diff

```
- 同样命令： git checkout SHA-1/TAG --master.d  可以使用某一历史来还原，不过这时需要进行git add,和git commit来确认提交

```
$ git lol -- master.d
* fb78979 1
* 257f3a2 modify master.d
*   d8e6e7d Merge branch 'branch_V0'
|\
| * 3b9ceaa (branch_V0) commit on branch_V0
* | 872942d (branch_merge) commit on branch_merge
* | 629aa7f second commit on master
|/
* 5154992 (tag: V0, tag: MASTER_FIRST) master.d first time commit

$ git checkout d8e6e7d -- master.d    //还原 SHA-1值为 d8e6e7d 的提交

```


## git reset 

将当前暂存区内容还原至某一历史的暂存区

```
# NAME
git-reset - Reset current HEAD to the specified state
```

- 使用当前最新历史记录的暂存区还原：
```
$ vim master.d
$ git add .                     //添加进暂存区

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git diff --cached             //查看暂存区与历史区差异
diff --git a/master.d b/master.d
index e7bd13e..e3bfabc 100644
--- a/master.d
+++ b/master.d
@@ -1,7 +1,7 @@
-111111111
+te111111111
 ef223eabc
-
+fef

Administrator@Just-pc MINGW32 /g/git_test/git_repo (master)
$ git reset -- master.d             //reset对master.d的操作
Unstaged changes after reset:
M       master.d

```
- 使用某一历史提交的暂存区的内容还原当前暂存区的内容

如：
```
$ git lol -- stage.txt  //文件准备，每次添加一行提交     
* 20373f0 (HEAD -> master) third commit
* 02ab411 second commit
* e99b96a first commit stage.txt

$ cat stage.txt        //当前文件中内容
first commit
second commit
third commit

$ vim stage.txt         //编辑，添加内容 'forth commit'
$ git add .             //添加进暂存区
$ git diff --cached     
diff --git a/stage.txt b/stage.txt
index 90dc5cb..e6d3e64 100644
--- a/stage.txt
+++ b/stage.txt
@@ -1,3 +1,4 @@
 first commit
 second commit
 third commit
+forth commit           //可以看到暂存区比历史区多了一行
```
执行 reset 操作

```
$ git reset 02ab411 -- stage.txt   //还原到某一时刻的stage，该时刻因该有两次add，即两行
Unstaged changes after reset:
M       stage.txt
$ git diff                         //工作区与暂存区区别
diff --git a/stage.txt b/stage.txt
index 56301e2..e6d3e64 100644
--- a/stage.txt
+++ b/stage.txt
@@ -1,2 +1,4 @@
 first commit
 second commit                    //当前工作区比 reset的暂存区多了两行 #与预期相同
+third commit
+forth commit

$ git diff --cached -- stage.txt    //暂存区与历史区区别
diff --git a/stage.txt b/stage.txt
index 90dc5cb..56301e2 100644
--- a/stage.txt
+++ b/stage.txt
@@ -1,3 +1,2 @@
 first commit
 second commit 
-third commit                    // 当前reset的暂存区比历史区少了一行 # 与预期相同

```

## git clean

git clean 会删除工作区中没有被git跟踪的文件：

```
$ git clean --help

##from git-clean Manual Page：
#name 
git-clean - Remove untracked files from the working tree
#synopsis 概要
git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] <path>…​

#DESCRIPTION 
Cleans the working tree by recursively removing files that are not under version control, starting from the current directory.
...
```
>默认，git clean会清除当前目录下的未跟踪文件；如果有 -x 参数，被忽略的文件也会被删除；
如果指定 <path>，则会删除指定目录下的未跟踪文件

常用参数命令 ：
命令| 说明
---|---
git -n| 显示将要做什么，而不真正删除，可以确认将要删除的文件
git -f| 强制删除，如文件夹
git -X| 大写的X，仅删除被git忽略的文件
git -x| 表示，git忽略的文件也会被删除，注意与 -X区别

For Example:
```
$ cat .gitignore         //查看忽略规则
*.[ab]
$ touch test.a test.b test.c test.d  //创建测试文件
$ git add test.d            //将test.d添加进 暂存区
```
根据以上操作，则 test.a 和 test.b文件是git忽略文件，test.c是未跟踪文件，test.d是 已跟踪文件

```
$ git clean -n                 //默认，仅删除未被跟踪的文件
Would remove test.c

$ git clean -x -n              //-x 小写，未被跟踪和忽略文件都会被删除
Would remove test.a
Would remove test.b
Would remove test.c

$ git clean -X -n              //-X 大写，仅删除被忽略文件
Would remove test.a
Would remove test.b

```

## git revert 

git revert 删除一些已提交的历史文件：

```
$ git clean --help
#NAME
git-revert - Revert some existing commits

#DESCRIPTION
Given one or more existing commits, revert the changes that the related patches introduce, and record some new commits that record them. This requires your working tree to be clean (no modifications from the HEAD commit).

```
>git revert 使用新的提交来覆盖已有的提交，当然要求将被覆盖的文件不要处于修改状态


```
$ git lol
* 54d2ffa (HEAD -> master) xxxcommit
* 074afd6 xxcommit
$ git revert 074afd6   //revert 074afd6提交

Administrator@Just-pc MINGW32 /g/git_test/git_repo/cleandir (master|REVERTING)
$ git status
On branch master
You are currently reverting commit 074afd6.
  (fix conflicts and run "git revert --continue")
  (use "git revert --abort" to cancel the revert operation)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

        both modified:   test.d

no changes added to commit (use "git add" and/or "git commit -a")

```
>提示使用 git revert --abort来撤销 revert操作或者 提交本次操作