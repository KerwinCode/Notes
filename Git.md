# Git学习笔记


## Git简介

### 相关知识

集中式版本控制系统：CVS、SVN

分布式版本控制系统：GIT


### 安装与配置

安装完成后需要做如下设置：

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

### 创建版本库

在特定目录中初始化Git仓库：`git init`

提交修改的步骤：

第一步，添加文件到仓库： `git add <file>`

第二步，提交文件到仓库： `git commit -m "introduce content"`


## 版本控制

### 版本回退

`git status` : 查看仓库当前状态

`git diff` : 查看仓库改动情况

`git log` : 查看提交历史，据此可以确定要回退到哪个版本

`git reflog` : 查看历史命令，据此确定要返回之后的哪个版本

`git reset --hard HEAD^` ： 回到上一个版本，上上一个版本可以写为`HEAD^^`，上100个版本可以写为`HEAD~100`

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从当前版本指向要回退的版本。


### 工作区与版本库

**工作区**：在电脑里能看到的目录，比如和当前git仓库对应的learngit文件夹就是一个工作区

**版本库**：工作区里的隐藏目录.git是git的版本库

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD。

我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，所以，现在，`git commit`就是往master分支上提交更改。

`git add`命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。提交之后，并且没有对工作区做任何修改，那么工作区就是“干净”的。


### 管理修改

Git跟踪并管理的是修改，而非文件。

每次修改，如果不add到暂存区，那就不会加入到commit中。


### 撤销修改

`git checkout -- file` : 撤销file在工作区的修改，分两种情况：

一种是file自修改后还没有被放到暂存区,撤销修改就回到和版本库一样的状态,即回到最近一次git commmit的状态；

另一种是file添加到暂存区后，又作了修改，撤销修改就回到和添加到暂存区时一样的状态，即回到最近一次git add时的状态。

**撤销修改的三个场景：**

场景1：当你改乱了工作区某个文件的内容（未添加到暂存区），想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，使用版本回退方法，不过前提是没有推送到远程库。


### 删除文件

从版本库中删除该文件，用命令`git rm`删掉，并且`git commit`；
如果删错了，可以使用 `git checkout -- file` 将误删的file文件恢复到最新版本。

`git checkout` 实质上是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以直接恢复。

**注意**：若出现误删时，只能恢复文件到最新版本，这样会丢失最近一次提交后修改的内容。


## 远程仓库

添加远程仓库的方法：

第一步，创建SSH Key（若已有SSH Key，可跳过）
`$ ssh-keygen -t rsa -C "youremail@example.com"`

第二步，在GitHub账户中添加SSH Key
在 Account settings 的 Add SSH Key 选项中填上Title，在Key文本框里粘贴id_rsa.pub文件的内容。


### 添加远程库

在GitHub上创建仓库后，将本地仓库和远程库关联起来，使用命令：

`git remote add origin git@server-name:path/repo-name.git`

关联后，使用命令 `git push -u origin master` 第一次推送master分支的所有内容；此后，每次本地提交后，只要有必要，就可以使用命令 `git push origin master` 推送最新修改；


### 从远程库克隆

要克隆一个仓库，首先必须知道仓库的地址，然后使用 `git clone` 命令克隆。

Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。


## 分支管理

在Git里，Head严格来说不是指向提交，而是指向分支，分支才是指向提交，Head指向的是当前分支。

在Git里主分支叫做master，Git鼓励创建分支，在新创建的分支上工作，再将该分支合并到主分支master上，最后可将创建的分支删掉。


### 创建和合并分支

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`

创建+切换分支：`git checkout -b <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`


### 解决冲突

出现合并冲突是因为两个分支修改了同一个文件的同一行或相邻的几行（怎么判断是不是相同的地方的不同修改啊，就是看这个修改的前后若干行要完全一致）

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，才能完成合并。

用`git log --graph`命令可以看到分支合并图。

用带参数的 `git log` 也可以看到分支的合并情况：

`git log --graph --pretty=oneline --abbrev-commit`


### 分支管理策略

合并分支时，加上 `--no-ff` 参数就可以用普通模式合并，合并后的历史有分支，

能看出来曾经做过合并，而 `fast forward` 合并就看不出来曾经做过合并。


### Bug分支

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场 `git stash` 一下，修复bug后，可以再回到工作现场。

工作现场可以用 `git stash list` 查看

恢复工作现场有两种方法：

一是用 `git stash apply` 恢复，但是恢复后，stash内容并不删除，需要用 `git stash drop` 来删除；

另一种方式是用 `git stash pop` ，恢复的同时把stash内容也删了：


### Feature分支

开发一个新feature，最好新建一个分支；

如果要丢弃一个没有被合并过的分支，可以通过 `git branch -D \<name\>` 强行删除。


### 多人协作

多人协作的工作模式通常是这样：

1.首先，可以试图用 `git push origin branch-name` 推送自己的修改；

2.如果推送失败，则因为远程分支比你的本地更新，需要先用 `git pull` 试图合并；

3.如果合并有冲突，则解决冲突，并在本地提交；

4.没有冲突或者解决掉冲突后，再用 `git push origin branch-name` 推送就能成功

如果 `git pull` 提示 `“no tracking information”` ，则说明本地分支和远程分支的链接关系没有创建，用命令 `git branch --set-upstream branch-name origin/branch-name` 。

查看远程库信息，使用`git remote -v`；

本地新建的分支如果不推送到远程，对其他人就是不可见的；

从本地推送分支，使用 `git push origin branch-name` ，如果推送失败，先用 `git pull` 抓取远程的新提交；

在本地创建和远程分支对应的分支，使用 `git checkout -b branch-name origin/branch-name` ，本地和远程分支的名称最好一致；

建立本地分支和远程分支的关联，使用 `git branch --set-upstream branch-name origin/branch-name` ；

从远程抓取分支，使用 `git pull` ，如果有冲突，要先处理冲突。


## 标签管理

Git的标签是版本库的快照，实质上它是指向某个commit的指针（跟分支很像，但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的。

tag是一个让人容易记住的有意义的名字，它跟某个commit绑在一起。


### 创建标签

`git tag <name>` 用于新建一个标签，默认为HEAD，也可以指定一个commit id；

`git tag -a <tagname> -m "blablabla..."` 可以指定标签信息；

`git tag -s <tagname> -m "blablabla..."` 可以用PGP签名标签；

`git tag` 可以查看所有标签。


### 操作标签

`git push origin <tagname>` : 可以推送一个本地标签；

`git push origin --tags` : 可以推送全部未推送过的本地标签；

`git tag -d <tagname>` : 可以删除一个本地标签；

`git push origin :refs/tags/<tagname>` : 可以删除一个远程标签。


## 自定义Git

让Git显示颜色 ： `git config --global color.ui true`


### 忽略特殊文件

忽略某些文件时，需要编写.gitignore；

.gitignore文件本身要放到版本库里，并且可以对.gitignore做版本管理。


### 配置别名

`$ git config --global alias.xxx 'xxx xxx'`

例如 `$ git config --global alias.unstage 'reset HEAD'`

配置Git的时候，加上 `--global` 是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。

每个仓库的Git配置文件都放在 `.git/config` 文件中；

当前用户的Git配置文件放在用户主目录下的一个隐藏文件 `.gitconfig` 中。


### 搭建Git服务器

[搭建Git服务器 - 廖雪峰的Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)

