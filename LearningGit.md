# Learning Git From LiaoXuefeng

## 附加内容

> **[一些Git操作的技巧](http://www.penglixun.com/tech/program/some_git_skills.html)**

### git stash

我们有时会遇到这样的情况，正在`分支a`上开发一半，然后`分支b`上发现Bug，需要马上处理。

这时候`分支a`上的修改怎么办呢，git add 是不行的，有的git客户端版本会提示还有add过的文件没提交不能切换分支，有的git客户端版本会把修改带到 `b分支`。

`git stash`就是解决这个问题，它把当前工作区的修改和 git add 的内容都保存到一个地方，然后 `git reset HEAD`，使工作区回到上一次提交，处于干净状态。然后就可以很放心的切到另外的`分支b`干活了。

```bash
git stash save “先给我保存一下，我要去别的分支修bug”
git stash list
git stash pop
git stash apply stash@{num}
```

### git rebase

有的时候我们在一个分支a开发的时候，master已经进入了很多修改，这时候如果把a的修改提交上去，可能就会跟主干有冲突，需要在主干解决冲突才能提交，这样比较难看。

这时候`git rebase`就有用了，`git rebase BRANCH_NAME`可以把`BRANCH_NAME分支`的修改带到当前分支来，这样当前分支就有了BRANCH_NAME分支的所有内容，这样在当前分支开发的内容提交以后不会跟BRANCH_NAME有冲突，冲突在当前分支就可以解决。

### git reset

可以取消已经提交的commit，一般我们只用`git reset HEAD^`。因为每个分支可能开发过程中为了保存过程以便回溯会有很多commit。

- **我们要求进入主干时，每个功能和bugfix只能有一个提交**
- **不管开发过程中有多少个commit，我们要求最终提交每个bugfix或feature只能有一个提交**

因此可以先用`git reset` 退回到最早的commit，然后把自己的修改最后`打包成一个commit`，再去跟主干合并。

利用这两个命令，我们可以很好的管理我们的MySQL开发。我们只有一个master分支作为主干，不允许在主干上直接开发。每个同学根据feature和bug的issue建立分支，然后在分支上开发。

因此每个同学完成开发后，一般需要以下步骤：

- `git reset`退到最早的commit
- `git stash save`保存一下自己的修改
- 然后`git checkout master`
- 之后`git pull`拉一下最新的主干
- 然后返回自己的分支，`git rebase master`，把当前分支推进到主干
- 最后`git stash pop`弹出修改
- 有冲突则在当前分支解决
- 再`git push`

### 链接引用

`[这是显示的名称](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743862006503a1c5bf5a783434581661a3cc2084efa000)`

[这是显示的名称](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743862006503a1c5bf5a783434581661a3cc2084efa000)

### 图片引用

`![这是图片别名](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907702917346729e9afbf4127b6dfbae9207af016000/0)`

![这是图片别名](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907702917346729e9afbf4127b6dfbae9207af016000/0)


## 初始化本地分支

```bash
git clone http://gitlab.greatopensource.com/abc/abc-traning.git
git remote rename origin upstream
git remote add origin http://gitlab.greatopensource.com/[your_name]/abc.git  
这个是你fork的自己的分支
```

### 本地开发步骤

```bash
git fetch
git checkout master
git merge origin/master
git checkout -b branchName      

# 做变更
git add -u
git add newFile

git commit -m "commit信息"
统一采用如下格式: 
第一行为对应的issue编号和题目，例如 Issue #2 如何调整非动态参数, 
第二行为空行，
从第三行开始写本次提交做的修改内容，可以考虑直接复制issue的内容

git push origin branchName     
该命令的输出中有一个git的链接，点击这个链接或在浏览器中打开这个链接会跳到 git 提 mr 的页面，然后设置 mr 的Assignee为自己并点上`Remove source branch`，然后提交
```

### 按照复审修改后的重新提交MR的方法

```bash
git add -u
git commit --amend
git push origin branchName --force
```

### 将本地分支更新到最新版本的方法

```bash
git fetch
git rebase upstream/master
[如果有冲突会报错并提示你那里有冲突，修改那个冲突文件] 

git add -u
git add newFile

git rebase --continue
然后重新提交
如果是第一次提交`git commit`，否则 `git commit --amend`
git push --force origin branchName 

也可以简写 -f 
git push -f origin branchName 
```

### 把本地多次提交合并成一个提交

```bash
git log > /tmp/logs
vim /tmp/logs 文件找到最新的master分支提交信息，例如:

commit a7f5981d1b842e645cd43e7efa9f300e4cae8cba
Author: zhenLEE <abc@lixyz.net>
Date:   Thu Apr 4 22:54:50 2019

使用 git reset a7f5981d1b842e645cd43e7efa9f300e4cae8cba 取消该master提交信息之后的所有commit
也可以使用前几位，git reset a7f5981d

然后 git add -u 和 git add newFile
git commit -m "balalala..."
git push --force origin branchName

也可以简写 -f 
git push -f origin branchName
```

## git reset 三种用法总结

- `git reset (–mixed) HEAD~1`
  - 回退一个版本,且会将暂存区的内容和本地已提交的内容全部恢复到未暂存的状态,
  - 不影响原来本地文件(未提交的也不受影响) 
- `git reset –soft HEAD~1`
  - 回退一个版本,不清空暂存区,将已提交的内容恢复到暂存区,
  - 不影响原来本地的文件(未提交的也不受影响) 
- `git reset –hard HEAD~1`
  - 回退一个版本,清空暂存区,将已提交的内容的版本恢复到本地,
  - 本地的文件也将被恢复的版本替换

## 删除 git 的分支

> <https://blog.csdn.net/taowuhua0505/article/details/80499540>

原理：推送一个空分支到远程分支，其实就相当于删除远程分支

命令：`$ git push origin 【空格】【冒号】【需要删除的分支名字`

比如github上有master和feature分支，我现在想着`删除feature分支`，命令如下：

```bash
$ git push origin :feature

# 或者常规方法
$ git push --delete origin feature 
$ git push -D origin feature 
```

## 初始设置

### 查看 & 设置用户信息

```bash
git config user.name
git config user.email

git config --global user.name 'zhenLEE'
git config --global user.email 'public@lixyz.net'
```

## 查看日志

```bash
git log

如果只需要简单显示：
git log --pretty=oneline

查看每次提交的改动：
git reflog
```

## 版本控制

### 返回某个版本

首先，Git必须知道当前版本是哪个版本<br>
在Git中，用`HEAD`表示`当前版本`，也就是`最新的提交`<br>
`上一个`版本就是`HEAD^`，`上上一个`版本就是`HEAD^^`<br>
当然`往上100个版本`写100个^比较容易数不过来，所以写成`HEAD~100`<br>

现在，把当前版本`回退到上一个版本`，可以使用`git reset`命令：

```bash
$ git reset --hard HEAD^
HEAD is now at e475afc add distributed

--hard参数有啥意义?
```

此时用git log再看看现在版本库的状态，<br>
最新的那个版本已经看不到了！<br>
好比你从21世纪坐时光穿梭机来到了19世纪，想再回去已经回不去了，肿么办？<br>

### 撤销还原

办法其实还是有的，只要上面的命令行窗口还没有被关掉，<br>
你就可以顺着往上找啊找啊，找到那个append GPL的commit id是1094adb...<br>
于是就可以指定回到未来的某个版本：<br>

```bash
$ git reset --hard 1094a
HEAD is now at 83b0afe append GPL
```

版本号没必要写全，前几位就可以了，Git会自动去找。<br>
当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了

---

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从指向append GPL：

git-head
![图片](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907594057a873c79f14184b45a1a66b1509f90b7a000/0)

### 恢复到任意一个

想恢复到新版本怎么办？找不到新版本的commit id怎么办？

在Git中，总是有后悔药可以吃的。当你用`$ git reset --hard HEAD^`回退到`add distributed`版本时，再想恢复到`append GPL`，就必须找到append GPL的`commit id`。Git提供了一个命令`git reflog`用来记录你的每一次命令：

```bash
$ git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file
```

终于舒了口气，从输出可知，`append GPL`的`commit id`是`1094adb`，现在，你又可以乘坐时光机回到未来了。

## 工作区，暂存区

### 工作区（Working Directory）

就是你在电脑里能看到的目录，比如我的 `abc.git` 文件夹就是一个工作区：`working-dir`

---

### 版本库（Repository）

工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，<br>
其中最重要的就是称为stage（或者叫index）的暂存区，<br>
还有Git为我们自动创建的第一个分支master，<br>
以及指向master的一个指针叫HEAD。<br>

![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907702917346729e9afbf4127b6dfbae9207af016000/0)

### 提交过程

前面讲了我们把文件往Git版本库里添加的时候，是分两步执行的：

1. 是用git add把文件添加进去，实际上就是把文件修改添加到暂存区；
2. 是用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支。

因为我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，<br>
所以,现在git commit就是往master分支上提交更改。

---

你可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。

### 过程测试

在工作区新建一个 license 文件，之后查看 git 状态：

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        license

nothing added to commit but untracked files present (use "git add" to track)
```

Git非常清楚地告诉我们，而LICENSE还从来没有被添加过，所以它的状态是Untracked。

现在，使用命令git add，把LICENSE添加后，用git status再查看一下：

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   license
```

此时的状态用图表示为：

![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907720458e56751df1c474485b697575073c40ae9000/0)

所以git add命令实际上就是把要提交的所有修改放到暂存区（Stage），
然后执行git commit就可以一次性把暂存区的所有修改提交到分支

---

此时进行提交后，查看状态

```bash
$ git commit -m '提交测试'
[master 93dda55] 提交测试
 1 file changed, 1 insertion(+)
 create mode 100644 license

$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

nothing to commit, working tree clean
```

如果提交后，没有进行其他修改，此时工作区就是干净的，<br>
版本库变成了这样，暂存区就没有任何内容了：

![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/0013849077337835a877df2d26742b88dd7f56a6ace3ecf000/0)

## Git管理的是修改，不是文件

按照以下操作过程修改文件：

1. 第一次修改
2. git add
3. git status
4. 第二次修改
5. git commit
6. git status

在 git add 后查看 status 发现有改动，

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   license
```

但是 commit 后查看 git status 发现第二次的修改没有提交

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   license
```

---

因为Git管理的是修改，<br>
当你用git add命令后，在工作区的第一次修改被放入暂存区，准备提交，<br>
但是，在工作区的第二次修改并没有放入暂存区，<br>
所以，git commit只负责把暂存区的修改提交了，<br>
也就是第一次的修改被提交了，第二次的修改不会被提交。<br>

---

提交后，用 git diff 命令查看工作区和版本库里面最新版本的区别：

```bash
$ git diff HEAD -- license
warning: LF will be replaced by CRLF in license.
The file will have its original line endings in your working directory.
diff --git a/license b/license
index 6262928..ecd2279 100644
--- a/license
+++ b/license
@@ -1,2 +1,3 @@
 add one document.a1b
 a1b
+c2d
```

那怎么提交第二次修改呢?<br>
你可以继续git add再git commit，<br>
也可以别着急提交第一次修改，<br>
先git add第二次修改，再git commit，<br>
就相当于把两次修改合并后一块提交了：<br>

1. 第一次修改
2. git add
3. 第二次修改
4. git add
5. git commit

每次修改，如果不用git add到暂存区，那就不会加入到commit中。

---

## 撤销修改

### 没有 add 到暂存区

如果修改完文件之后，需要撤销，手动恢复到上一个版本

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   license

no changes added to commit (use "git add" and/or "git commit -a")
```

Git会告诉你，git checkout -- file可以丢弃工作区的修改：

> git checkout -- license

命令`git checkout -- readme.txt`意思就是，把readme.txt文件在工作区的修改全部撤销, 让这个文件回到最近一次git commit或git add时的状态

`git checkout -- file` 命令中的 `--` 很重要，`没有 --` ，就变成了`切换到另一个分支`的命令，我们在后面的分支管理中会再次遇到 git checkout 命令

### 已经 add 到暂存区

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   license
```

Git同样告诉我们，用命令`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区：

```bash
$ git reset HEAD license
Unstaged changes after reset:
M       license

$ cat license
first
second
third
one
two
1
2
```

此时查看状态，暂存区是干净的，工作区有修改：

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   license

no changes added to commit (use "git add" and/or "git commit -a")
```

此时可以丢弃工作区的修改

```bash
$ git checkout -- license

此时查看，最新添加修改的 1 2 已经没有了
$ cat license
first
second
third
one
two
```

### 小结

1. 当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`
2. 当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步:
   1. 第一步用命令`git reset HEAD <file>`，就回到了1，
   2. 第二步按1操作，`git checkout -- file`
3. 已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

## 删除文件

在 Git 中，删除也是修改

如果你 add 了一个文件，之后直接 rm file 进行了删除，git status 就会显示已经删除了一个文件

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    license

no changes added to commit (use "git add" and/or "git commit -a")
```

### 1、确实需要删除文件

如果要从版本库中删除该文件，那就用命令`git rm`删掉，并且`git commit`

```bash
$ git rm license
rm 'license'

$ git commit -m 'remove license'
[master 363fa32] remove license
 1 file changed, 5 deletions(-)
 delete mode 100644 license
```

> 先手动删除文件，然后使用`git rm <file>`和`git add <file>`效果是一样的

### 2、如果是不小心删错了

因为版本库里还有呢，<br>
所以可以很轻松地把误删的文件恢复到最新版本：

> git checkout -- license

git checkout 其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以`一键还原`

```bash
$ rm -f license

$ git checkout -- license

$ ls
license  readme.txt

$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   license
```

### 小结

命令 git rm 用于删除一个文件。<br>
如果一个文件已经被提交到版本库，<br>
那么你永远不用担心误删。<br>
但是要小心，你只能恢复文件到最新版本，<br>
你会丢失最近一次提交后你修改的内容。<br>

## 关联到远程仓库

### GitHub 创建远程仓库

> [查看原文](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013752340242354807e192f02a44359908df8a5643103a000)

### 本地关联远程仓库

> git remote add origin git@github.com:michaelliao/learngit.git

远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库

### 解除关联远程仓库

> git remote remove origin

### 把本地库的内容推送到远程库上

> git push -u origin master

把本地库的内容推送到远程，用git push命令，<br>
实际上是把当前分支master推送到远程。<br>

由于远程库是空的，<br>
我们第一次推送master分支时，加上了-u参数，<br>
Git不但会把本地的master分支内容推送的远程新的master分支，<br>
还会把本地的master分支和远程的master分支关联起来，<br>
在以后的推送或者拉取时就可以简化命令。<br>

从现在起，只要本地作了提交，就可以通过命令：

> git push origin master

把本地master分支的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库

### 注意

- 如果提示 "远程的原本落后与整个版本"，
- 那么你只能 `git push -u origin master -f` 强制 push
- 其实一般开发的步骤，git init 之后我们都需要从远端pull，然后再做更改，再push

### 小结

要关联一个远程库，使用命令

> git remote add origin git@server-name:path/repo-name.git

关联后，第一次推送master分支的所有内容；

> git push -u origin master

此后，每次本地提交后，只要有必要，就可以推送最新修改；

> git push origin master

分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，<br>
也就是有没有联网都可以正常工作，<br>
而SVN在没有联网的时候是拒绝干活的！<br>
当有网络的时候，再把本地提交推送一下就完成了同步。<br>

## 从远程库克隆

上一节讲了先有本地库，后有远程库的时候，如何关联远程库。

现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。

### GitHub 创建远程库

> <https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375233990231ac8cf32ef1b24887a5209f83e01cb94b000>

现在，远程库已经准备好了

### git clone克隆一个本地库

> $ git clone git@github.com:michaelliao/gitskills.git

---

如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了。<br>
GitHub给出的地址不止一个，还可以用<https://github.com/michaelliao/gitskills.git>这样的地址。<br>
这是因为Git支持多种协议，默认的git://使用ssh，但也可以使用https等其他协议。<br>
使用https除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，<br>
但是在某些只开放http端口的公司内部就无法使用ssh协议而只能用https。

### 小结

要克隆一个仓库，首先必须知道仓库的地址，然后使用git clone命令克隆。

Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。

## 分支管理

> <https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743862006503a1c5bf5a783434581661a3cc2084efa000>

分支在实际中有什么用呢？<br>
假设你准备开发一个新功能，但是需要两周才能完成，<br>
第一周你写了50%的代码，<br>
如果立刻提交，由于代码还没写完，不完整的代码库会导致别人不能干活了。<br>
如果等代码全部写完再一次提交，又存在丢失每天进度的巨大风险。<br>
<br>
现在有了分支，就不用怕了。<br>
你创建了一个属于你自己的分支，<br>
别人看不到，还继续在原来的分支上正常工作，<br>
而你在自己的分支上干活，想提交就提交，<br>
直到开发完毕后，再一次性合并到原来的分支上，<br>
这样，既安全，又不影响别人工作。<br>
<br>
其他版本控制系统如SVN等都有分支管理，<br>
但是用过之后你会发现，<br>
这些版本控制系统创建和切换分支比蜗牛还慢，简直让人无法忍受，<br>
结果分支功能成了摆设，大家都不去用。<br>
<br>
但Git的分支是与众不同的，<br>
无论创建、切换和删除分支，Git在1秒钟之内就能完成，<br>
无论你的版本库是1个文件还是1万个文件。<br>

## 创建与合并分支

> [相关介绍](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840038939c291467cc7c747b1810aab2fb8863508000)

### 创建并切换到dev分支

```bash
$ git checkout -b dev
Switched to a new branch 'dev'
```

git checkout 命令加上 `-b` 参数表示创建并切换，相当于以下两条命令：

```bash
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
```

然后，用git branch命令查看当前分支：

```bash
$ git branch
* dev
&ensp; master
```

git branch 命令会列出所有分支，`当前分支`前面会标一个`*`号。

然后，我们就可以在dev分支上正常提交，比如对readme.txt做个修改，加上一行：

> Creating a new branch is quick.

然后提交：

```bash
$ git add readme.txt
$ git commit -m "branch test"
[dev b17d20e] branch test
 1 file changed, 1 insertion(+)
```

### 切换回master分支：

```bash
$ git checkout master
Switched to branch 'master'
```

切换回master分支后，再查看一个readme.txt文件，刚才添加的内容不见了！因为那个提交是在dev分支上，而master分支此刻的提交点并没有变：

![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384908892295909f96758654469cad60dc50edfa9abd000/0)

### 把dev分支合并到master分支

```bash
$ git merge dev
Updating d46f35e..b17d20e
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```

git merge命令用于合并指定分支到当前分支。合并后，再查看readme.txt的内容，就可以看到，和dev分支的最新提交是完全一样的。

注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。

当然，也不是每次合并都能Fast-forward，我们后面会讲其他方式的合并。

### 合并完成后，可以删除dev分支了

```bash
$ git branch -d dev
Deleted branch dev (was b17d20e).
```

删除后，查看branch，就只剩下master分支了：

```bash
$ git branch
* master
```

因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，<br>
合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。

### 小结

Git鼓励大量使用分支：

- 查看分支：`git branch`
- 创建分支：`git branch <name>`
- 切换分支：`git checkout <name>`
- 创建+切换分支：`git checkout -b <name>`
- 合并某分支到当前分支：`git merge <name>`
- 删除分支：`git branch -d <name>`

## 解决冲突

### 准备feature1分支，继续分支开发：

```bash
$ git checkout -b feature1
Switched to a new branch 'feature1'
```

### 修改readme.txt最后一行

Creating a new branch is quick AND simple.

### 在feature1分支上提交

```bash
$ git add readme.txt

$ git commit -m "AND simple"
[feature1 14096d0] AND simple
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### 切换到master分支

```bash
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
```

Git还会自动提示我们当前master分支比远程的master分支要超前1个提交。

在master分支上把readme.txt文件的最后一行改为：

Creating a new branch is quick & simple.

提交：

```bash
$ git add readme.txt
$ git commit -m "& simple"
[master 5dc6824] & simple
 1 file changed, 1 insertion(+), 1 deletion(-)
```

现在，master分支和feature1分支各自都分别有新的提交，变成了这样：

![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384909115478645b93e2b5ae4dc78da049a0d1704a41000/0)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，我们试试看：

```bash
git merge feature1
Auto-merging license
CONFLICT (content): Merge conflict in license
Automatic merge failed; fix conflicts and then commit the result.
```

果然冲突了！<br>
Git告诉我们，readme.txt文件存在冲突，必须手动解决冲突后再提交。<br>
git status 也可以告诉我们冲突的文件：<br>

```bash
$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

        both modified:   license

no changes added to commit (use "git add" and/or "git commit -a")
```

此时查看 license 文件的内容

```bash
$ cat license
a
b
c
<<<<<< HEAD
add two maste
=======
add one
>>>>>> feature1
```

### 处理冲突

Git用 `<<<<<<<，=======，>>>>>>>` 标记出不同分支的内容，我们修改如下后保存：

> add two master

再提交：

```bash
$ git add license

$ git commit -m 'license';
[master a9895a4] license
```

现在的图例：

![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/00138490913052149c4b2cd9702422aa387ac024943921b000/0)

用带参数的git log也可以看到分支的合并情况：

```bash
 git log --graph --pretty=oneline --abbrev-commit
*   a9895a4 (HEAD -> master) license
|\
| * 7aa3946 (feature1) two
* | 6c467ef simple
|/
* dcf01be test
* 363fa32 remove license
* 5932532  Changes to be committed:     new file:   license
* da2ef27  Changes to be committed:     deleted:    license
* 1d0f5cb  Changes not staged for commit:       deleted:    license
* 4567eb8 c2d
* 93dda55 提交测试
* 777f979 add distributed
* 4bf29a3 the first commit
```

### 删除 feature1 分支

> git branch -d feature1
> Deleted branch feature1 (was 7aa3946).

### 小结

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。<br>
解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。<br>
用git log --graph命令可以看到分支合并图。

## 分支管理策略

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。<br>
如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。<br>
下面我们实战一下`--no-ff`方式的`git merge`：

首先，仍然创建并切换dev分支：

```bash
$ git checkout -b dev
Switched to a new branch 'dev'
```

修改 license，并提交一个新的 commit

```bash
$ vi license

$ git add license

$ git commit -m 'dev one'
[dev 22d0929] dev one
 1 file changed, 1 insertion(+)
```

现在切换回 master

```bash
$ git checkout master
Switched to branch 'master'
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

$ git status;
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

nothing to commit, working tree clean
```

准备合并 dev 分支，使用 `--no-ff`，表示禁用 fast forward，因为本次合并要创建一个新的commit，所以加上-m`参数，把commit描述写进去。

```bash
$ git merge --no-ff -m 'dev two no ff' dev
Merge made by the 'recursive' strategy.
 license | 1 +
 1 file changed, 1 insertion(+)
```

合并后，查看 分支历史

```bash
$ git log --graph --pretty=oneline --abbrev-commit
*   bd73cbe (HEAD -> master) dev two no ff
|\
| * 22d0929 (dev) dev one
|/
*   a9895a4 license
|\
| * 7aa3946 two
* | 6c467ef simple
|/
* dcf01be test
* 363fa32 remove license
* 5932532  Changes to be committed:     new file:   license
* da2ef27  Changes to be committed:     deleted:    license
* 1d0f5cb  Changes not staged for commit:       deleted:    license
* 4567eb8 c2d
* 93dda55 提交测试
* 777f979 add distributed
* 4bf29a3 the first commit
```

可以看到，不使用Fast forward模式，merge后就像这样：
![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384909222841acf964ec9e6a4629a35a7a30588281bb000/0)

### 分支策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

1. master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
2. 干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
3. 你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

所以，团队合作的分支看起来就像这样：
![图例](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384909239390d355eb07d9d64305b6322aaf4edac1e3000/0)

### 小结

Git分支十分强大，在团队开发中应该充分应用。

合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。

## Bug 分支

软件开发中，bug就像家常便饭一样。有了bug就需要修复，<br>
在Git中，由于分支是如此的强大，<br>
所以每个bug都可以通过一个新的临时分支来修复，<br>
修复后，合并分支，然后将临时分支删除。<br>
<br>
当你接到一个修复一个代号101的bug的任务时，<br>
很自然地，你想创建一个分支issue-101来修复它，<br>
但是，当前正在dev上进行的工作还没有提交：<br>
<br>
并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。<br>
但是，必须在两个小时内修复该bug，怎么办？<br>

幸好，Git还提供了一个stash功能，可以把当前工作现场`储藏`起来，等以后恢复现场后继续工作：

> git stash

现在，用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），<br>
因此可以放心地创建分支来修复bug。<br>
<br>
首先确定要在哪个分支上修复bug，<br>
假定需要在master分支上修复，就从master创建临时分支：<br>

```bash
$ git checkout master
Already on 'master'
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```

现在修复bug，然后提交：

```bash
$ git add license

$ git commit -m 'fix bug 101'
[issue-101 9e04c3c] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：

```bash
$ git checkout master
Switched to branch 'master'
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)

$ git merge --no-ff -m 'merged 101 bug' issue-101
Merge made by the 'recursive' strategy.
 license | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### 修复完成后，回到 dev 分支

此时工作区是干净的，刚才的工作现场存到哪去了？用`git stash list`命令看看：

```bash
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

1. 用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；
2. 用git stash pop，恢复的同时把stash内容也删了：

你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：

> git stash apply stash@{0}

### 小结

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场。

## features 分支

软件开发中，总有无穷无尽的新的功能要不断添加进来。

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，<br>
所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

现在，你终于接到了一个新任务：<br>
开发代号为Vulcan的新功能，该功能计划用于下一代星际飞船。

于是准备开发：

```bash
$ git checkout -b feature-vulcan
Switched to a new branch 'feature-vulcan'
```

5分钟后，开发完毕：

```bash
$ git add vulcan.py

$ git status
On branch feature-vulcan
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   vulcan.py

$ git commit -m "add feature vulcan"
[feature-vulcan 287773e] add feature vulcan
 1 file changed, 2 insertions(+)
 create mode 100644 vulcan.py
```

切回dev，准备合并：`git checkout dev`<br>
一切顺利的话，feature分支和bug分支是类似的，合并，然后删除。

但是！就在此时，接到上级命令，因经费不足，新功能必须取消！<br>
虽然白干了，但是这个包含机密资料的分支还是必须就地销毁：

```bash
$ git branch -d feature-vulcan
error: The branch 'feature-vulcan' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature-vulcan'.
```

销毁失败。<br>
Git友情提醒，feature-vulcan分支还没有被合并，<br>
如果删除，将丢失掉修改，<br>
如果要强行删除，需要使用大写的-D参数。

现在我们强行删除：

```bash
$ git branch -D feature-vulcan
Deleted branch feature-vulcan (was 287773e).
```

终于删除成功！

### 小结

开发一个新feature，最好新建一个分支；<br>
如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。

## 多人协作

当你从远程仓库克隆时，<br>
实际上Git自动把本地的master分支和远程的master分支对应起来了，<br>
并且，远程仓库的默认名称是origin。

### 查看远程库的信息

`git remote`或者 `git remote -v` 显示更详细的信息：

```bash
$ git remote
origin

$ git remote -v
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
```

上面显示了可以抓取和推送的origin的地址。如果没有推送权限，就看不到push的地址。

### 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。

推送时，要指定本地分支，<br>
这样，Git就会把该分支推送到远程库对应的远程分支上：

> git push origin master

如果要推送其他分支，比如dev，就改成：

> git push origin dev

但是，并不是一定要把本地分支往远程推送，<br>
那么，哪些分支需要推送，哪些不需要呢？

- master分支是主分支，因此要时刻与远程同步；
- dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
- feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

总之，就是在Git中，分支完全可以在本地自己藏着玩，是否推送，视你的心情而定！

### 抓取分支

多人协作时，大家都会往master和dev分支上推送各自的修改。

现在，模拟一个你的小伙伴，可以在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下克隆：

```bash
git clone git@github.com:michaelliao/learngit.git
```

当你的小伙伴从远程库clone时，<br>
默认情况下，你的小伙伴只能看到本地的master分支。<br>
可以用 `git branch` 命令看看：

```bash
$ git branch
* master
```

现在，你的小伙伴要在dev分支上开发，就必须创建远程origin的dev分支到本地，<br>
于是他用这个命令创建本地dev分支：

```bash
git checkout -b dev origin/dev
```

现在，他就可以在dev上继续修改，<br>
然后，时不时地把dev分支push到远程：

```bash
git add env.txt
git commit -m "add env"
git push origin dev
```

你的小伙伴已经向origin/dev分支推送了他的提交，<br>
而碰巧你也对同样的文件作了修改，并试图推送：

```bash
git add env.txt
git commit -m "add new env"
git push origin dev
```

推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，

解决办法也很简单，<br>
Git已经提示我们，先用git pull把最新的提交从origin/dev抓下来，<br>
然后，在本地合并，解决冲突，再推送：

> git pull

git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接，<br>
根据提示，设置dev和origin/dev的链接：

```bash
$ git branch --set-upstream-to=origin/dev dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
```

再pull：

```bash
$ git pull
Auto-merging env.txt
CONFLICT (add/add): Merge conflict in env.txt
Automatic merge failed; fix conflicts and then commit the result.
```

这回git pull成功，但是合并有冲突，需要手动解决，<br>
解决的方法和分支管理中的解决冲突完全一样。解决后，提交，再push：

```bash
$ git commit -m "fix env conflict"
[dev 57c53ab] fix env conflict

$ git push origin dev
```

因此，多人协作的工作模式通常是这样：

- 首先，可以试图用git push origin \<branch-name>推送自己的修改；
- 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；
- 如果合并有冲突，则解决冲突，并在本地提交；
- 没有冲突或者解决掉冲突后，再用git push origin \<branch-name>推送就能成功！

如果`git pull`提示`no tracking information`，<br>
则说明本地分支和远程分支的链接关系没有创建，

用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

### 小结

- 查看远程库信息，使用git remote -v；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用git push origin branch-name，如果推送失败，先用git pull抓取远程的新提交；
- 在本地创建和远程分支对应的分支，
  - 使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用git branch --set-upstream branch-name origin/branch-name；
- 从远程抓取分支，使用git pull，如果有冲突，要先处理冲突。

## Rebase

在上一节我们看到了，<br>
多人在同一个分支上协作时，很容易出现冲突。<br>
即使没有冲突，后push的童鞋不得不先pull，在本地合并，然后才能push成功。

每次合并再push后，分支变成了这样：

### **本章未看**

> 网页
> <https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0015266568413773c73cdc8b4ab4f9aa9be10ef3078be3f000>

## 标签管理

发布一个版本时，我们通常先在版本库中打一个标签（tag），<br>
这样，就唯一确定了打标签时刻的版本。<br>
将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。<br>
所以，标签也是版本库的一个快照。

Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针。<br>
跟分支很像对不对？但是分支可以移动，标签不能移动，<br>
所以，创建和删除标签都是瞬间完成的。

---

Git有commit，为什么还要引入tag？

> “请把上周一的那个版本打包发布，commit号是6a5819e...”
> 
> “一串乱七八糟的数字不好找！”
> 
> 如果换一个办法：
> 
> “请把上周一的那个版本打包发布，版本号是v1.2”
> 
> “好的，按照tag v1.2查找commit就行！”

所以，tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起。

### 创建标签

在Git中打标签非常简单，首先，切换到需要打标签的分支上：

```bash
$ git branch
* dev
  master
$ git checkout master
Switched to branch 'master'
```

然后，敲命令`git tag <name>`就可以打一个新标签：

```bash
git tag v1.0
```

可以用命令`git tag`查看所有标签：

```bash
$ git tag
v1.0
```

默认标签是打在最新提交的commit上的。<br>
有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打，怎么办？

方法是找到历史提交的 `commit id`，然后打上就可以了：

```bash
$ git log --pretty=oneline --abbrev-commit
12a631b (HEAD -> master, tag: v1.0, origin/master) merged bug fix 101
4c805e2 fix bug 101
e1e9c68 merge with no-ff
f52c633 add merge
cf810e4 conflict fixed
5dc6824 & simple
14096d0 AND simple
b17d20e branch test
d46f35e remove test.txt
b84166e add test.txt
519219b git tracks changes
e43a48b understand how stage works
1094adb append GPL
e475afc add distributed
eaadf4e wrote a readme file
```

比方说要对`add merge`这次提交打标签，它对应的`commit id`是`f52c633`，敲入命令：

`git tag v0.9 f52c633`

再用命令git tag查看标签：

```bash
$ git tag
v0.9
v1.0
```

注意，标签不是按时间顺序列出，而是按字母排序的。<br>
可以用`git show <tagname>`查看标签信息：

```bash
$ git show v0.9
commit f52c63349bc3c1593499807e5c8e972b82c8f286 (tag: v0.9)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:56:54 2018 +0800

    add merge

diff --git a/readme.txt b/readme.txt
...
```

可以看到，v0.9确实打在add merge这次提交上。

还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：

`git tag -a v0.1 -m "version 0.1 released" 1094adb`

用命令`git show <tagname>`可以看到说明文字：

```bash
$ git show v0.1
tag v0.1
Tagger: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 22:48:43 2018 +0800

version 0.1 released

commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (tag: v0.1)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:06:15 2018 +0800

    append GPL

diff --git a/readme.txt b/readme.txt
...
```

注意：标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。

### 小结

1. 命令`git tag <tagname>`用于新建一个标签，`默认为HEAD`，也可以`指定一个commit id`；
2. 命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息；

### 删除标签

```bash
$ git tag -d v0.1
Deleted tag 'v0.1' (was f15b0dd)
```

因为创建的标签都只存储在本地，不会自动推送到远程。<br>
所以，打错的标签可以在本地安全删除。

如果要推送某个标签到远程，<br>
使用命令`git push origin <tagname>`

```bash
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v1.0 -> v1.0
```

或者，一次性推送全部尚未推送到远程的本地标签：

```bash
$ git push origin --tags
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v0.9 -> v0.9
```

如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：

```bash
$ git tag -d v0.9
Deleted tag 'v0.9' (was f52c633)
```

然后，从远程删除。删除命令也是push，但是格式如下：

```bash
$ git push origin :refs/tags/v0.9
To github.com:michaelliao/learngit.git
&ensp;- [deleted]         v0.9
```

要看看是否真的从远程库删除了标签，可以登陆GitHub查看。

### 小结

- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

## 自定义 git

让 Git 适当地显示不同的颜色

`git config --global color.ui true`

### 忽略特殊文件

有些时候，你必须把某些文件放到Git工作目录中，但又不能提交它们，<br>
比如保存了数据库密码的配置文件啦，等等，<br>
每次git status都会显示Untracked files ...<br>
<br>
解决方法：<br>
在Git工作区的根目录下创建一个特殊的.gitignore文件，<br>
然后把要忽略的文件名填进去，Git就会自动忽略这些文件。<br>
<br>
不需要从头写.gitignore文件，GitHub已经为我们准备了各种配置文件，<br>
只需要组合一下就可以使用了。<br>
所有配置文件可以直接在线浏览：<br>
<https://github.com/github/gitignore><br>

忽略文件的原则是：

- 忽略操作系统自动生成的文件，比如缩略图等；
- 忽略编译生成的中间文件、可执行文件等，
  - 也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，
  - 比如Java编译产生的.class文件；
- 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

举个例子：

- 假设你在Windows下进行Python开发，Windows会自动在有图片的目录下生成隐藏的缩略图文件，
- 如果有自定义目录，目录下就会有Desktop.ini文件，
- 因此你需要忽略Windows自动生成的垃圾文件：

```bash
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini
```

然后，继续忽略Python编译产生的.pyc、.pyo、dist等文件或目录：

```bash
# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build
```

加上你自己定义的文件，最终得到一个完整的.gitignore文件，内容如下：

```bash
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini

# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build

# My configurations:
db.ini
deploy_key_rsa
```

最后一步就是把.gitignore也提交到Git，就完成了！

当然检验`.gitignore`的标准是`git status`命令是不是说`working directory clean`

windows 在创建文件时，会提示没有文件名。

解决方法：

- 在资源管理创建文件时，文件命名“.gitignore.”，
  - 注意结尾有个.号，回车确认时系统会自动存成.gitignore。
- 可以在文本编辑器里“保存”或者“另存为”就可以把文件保存为`.gitignore`

### 忽略文件导致的问题

有些时候，你想添加一个文件到Git，但发现添加不了，原因是这个文件被`.gitignore`忽略了：

```bash
$ git add App.class
The following paths are ignored by one of your .gitignore files:
App.class
Use -f if you really want to add them.
```

如果你确实想添加该文件，可以用-f强制添加到Git：
> git add -f App.class

或者你发现，可能是.gitignore写得有问题，需要找出来到底哪个规则写错了，可以用`git check-ignore`命令检查：

```bash
$ git check-ignore -v App.class
.gitignore:3:*.class    App.class
```

Git会告诉我们，`.gitignore`的第`3`行规则忽略了该文件，于是我们就可以知道应该修订哪个规则。

### 小结

- 忽略某些文件时，需要编写.gitignore；
- .gitignore文件本身要放到版本库里，并且可以对.gitignore做版本管理！

### 网友评论

指定忽略某个路径下的所有文件

```bash
path-to-ignore/
/xxx/xxx/*
```

## 配置别名

我们只需要敲一行命令，以后st就表示status：

`git config --global alias.st status`

当然还有别的命令可以简写，如：

- co表示checkout，
- ci表示commit，
- br表示branch：

```bash
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
```

以后提交就可以简写成：

`git ci -m "bala bala bala..."`

--global参数是全局参数，也就是这些命令在这台电脑的所有Git仓库下都有用。

在撤销修改一节中，我们知道，<br>
命令`git reset HEAD file`可以把暂存区的修改撤销掉`unstage`，重新放回工作区。<br>
既然是一个unstage操作，就可以配置一个unstage别名：<br>

> `git config --global alias.unstage 'reset HEAD'`

当你敲入命令：
> `git unstage test.py`

实际上Git执行的是：

> `git reset HEAD test.py`

配置一个git last，让其显示最后一次提交信息：

> `git config --global alias.last 'log -1'`

这样，用git last就能显示最近一次的提交：

```bash
$ git last
commit adca45d317e6d8a4b23f9811c3d7b7f0f180bfe2
Merge: bd6ae48 291bea8
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Thu Aug 22 22:49:22 2013 +0800

    merge & fix hello.py
```

甚至还有人丧心病狂地把lg配置成了：

> `git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"`

来看看git lg的效果：
> git -lg
![lg](https://cdn.liaoxuefeng.com/cdn/files/attachments/00138492662982594cbd1a942114472aeeb5f0a502faed1000/0)

---

为什么不早点告诉我？别激动，咱不是为了多记几个英文单词嘛！

### 配置文件

- 配置Git的时候，加上`--global`是针对当前用户起作用的
- 如果不加，那只针对当前的仓库起作用。

配置文件放哪了？每个仓库的Git配置文件都放在`.git/config`文件中：

```bash
$ cat .git/config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
    ignorecase = true
    precomposeunicode = true
[remote "origin"]
    url = git@github.com:michaelliao/learngit.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = origin
    merge = refs/heads/master
[alias]
    last = log -1
```

别名就在`[alias]`后面，要删除别名，直接把对应的行删掉即可。

而当前用户的Git配置文件放在用户主目录下的一个隐藏文件`.gitconfig`中：

```bash
$ cat .gitconfig
[alias]
    co = checkout
    ci = commit
    br = branch
    st = status
[user]
    name = Your Name
    email = your@email.com
```

配置别名也可以直接修改这个文件，如果改错了，可以删掉文件重新通过命令配置。

### 小结

给Git配置好别名，就可以输入命令时偷个懒。我们鼓励偷懒。

## 搭建Git服务器

> [Git-Server](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)
