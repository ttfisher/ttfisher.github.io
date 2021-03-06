---
title: GitHub博客平台搭建（四）：谈一谈越来越火的Git
categories:
  - 多元化技能储备
  - Git&GitHub
tags:
  - GitHub
abbrlink: f743d1c
date: 2018-01-29 17:17:00
---
【引言】为什么要写这么一篇总结呢？一来是为了圆了前面那一篇的说辞；二来是因为零零散散的接触Git也有些时日了，谈起Git竟不知道从何谈起，说不出个一二三来，所以，抽点时间整理一下，也纯当给自己的学习做个总结吧！（部分资源参考自网络）
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-07-29-01.jpg" width="55%"/></div>

<!-- more -->

# Git是什么？
&emsp;&emsp;传统的最常用的版本控制工具SVN，它和Git有什么区别呢？从下图就可以一窥究竟，最主要的区别就是SVN是集中式的，而Git则是分布式的，而且Git是目前世界上最先进的分布式版本控制系统（没有之一）。
&emsp;&emsp;Git完全不依赖于中心服务器来保存文件的旧版本，任何一台机器都可以有一个本地的版本控制系统（实际上就是对应硬盘上的文件），我们也可以称之为仓库（repository）；但是团队开发多人协作的话，我们就需要一个独立的线上仓库，用来同步所有成员的代码变更，而GitHub就是这个功能。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2019/2019-05-16-01.jpg" width="600"/></div>

# Git的诞生趣事
&emsp;&emsp;很多人都知道，Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。Linus虽然创建了Linux，但Linux的壮大是靠全世界热心的志愿者参与的，这么多人在世界各地为Linux编写代码，那Linux的代码是如何管理的呢？事实是，在2002年以前，世界各地的志愿者把源代码文件通过diff的方式发给Linus，然后由Linus本人通过手工方式合并代码！
&emsp;&emsp;你也许会想，为什么Linus不把Linux代码放到版本控制系统里呢？不是有CVS、SVN这些免费的版本控制系统吗？因为Linus坚定地反对CVS和SVN，这些集中式的版本控制系统不但速度慢，而且必须联网才能使用。有一些商用的版本控制系统，虽然比CVS、SVN好用，但那是付费的，和Linux的开源精神不符。
&emsp;&emsp;不过，到了2002年，Linux系统已经发展了十年了，代码库之大让Linus很难继续通过手工方式管理了，社区的弟兄们也对这种方式表达了强烈不满，于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。安定团结的大好局面在2005年就被打破了，原因是Linux社区牛人聚集，不免沾染了一些梁山好汉的江湖习气。开发Samba的Andrew试图破解BitKeeper的协议（这么干的其实也不只他一个），被BitMover公司发现了（监控工作做得不错！），于是BitMover公司怒了，要收回Linux社区的免费使用权。
&emsp;&emsp;Linus可以向BitMover公司道个歉，保证以后严格管教弟兄们，嗯，这是不可能的。实际情况是这样的：Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git！一个月之内，Linux系统的源码已经由Git管理了！牛是怎么定义的呢？大家可以体会一下。Git迅速成为最流行的分布式版本控制系统，尤其是2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub，包括jQuery，PHP，Ruby等等。
&emsp;&emsp;历史就是这么偶然，如果不是当年BitMover公司威胁Linux社区，可能现在我们就没有免费而超级好用的Git了。
> 注：本段落内容摘自大牛廖雪峰的Git教程，感谢有许许多多这么有奉献精神的大牛。

# Git的安装
&emsp;&emsp;Windows安装下载地址：https://git-scm.com/downloads ；下载完成后在本机安装（具体下载什么版本就看电脑操作系统和位数了）；一路Next，安装完成后就会有一个Git Bash的程序，正常运行即表示安装成功了；安装完成后，按照下面的命令配置个人的用户名和邮箱（对了，在这之前最好是先注册一个GitHub的账号，包括安装和初始化配置的具体操作都可参考前一篇文章）。
&emsp;&emsp;常用的Linux环境安装的话，一般通过 `sudo apt-get install git` 这个命令就可以完成；如果有些特殊的系统版本，也可以直接通过官网的源码包安装（步骤：1. 从Git官网下载源码到本机并解压，2. 到解压目录执行： `./config` 、 `make` 、`sudo make install` ）。

# Git的配置
&emsp;&emsp;通常，我们需要做如下的配置，这里的`git config`命令就是用来完成配置功能的，后面的global选项则是指定这是全局配置（当然也可以不选用）：
```bash
# 填写你的github用户名
$ git config --global user.name "your user name" 

# 填写你的github注册邮箱
$ git config --global user.email "your email address" 
```

# Git的工作流程
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2019/2019-05-16-02.jpg" width="55%"/></div>
&emsp;&emsp;不管是独立使用还是团队协作，使用Git作版本管理的操作都要遵循一个既定的流程，而上面这张图就很好地给我们展示了Git的工作流程：
- 首先肯定要创建一个工作目录，一般都是克隆Git仓库的资源作为自己的工作目录
- 资源下载到本地，肯定离不了增删改，这时就需要在克隆的资源上进行添加、修改、删除文件的操作了
- 因为是多人协作，难免你修改的过程中别人提交了修改，那么你在修改完提交之前肯定要再更新一下资源确保不会覆盖别人修改的内容
- 在提交前查看自己的修改，这个也是确保提交准确性的
- 在确认前面的操作都OK之后，提交修改到仓库
- 在修改完成后，若发现错误或遗漏，可以撤回提交并再修改过后重新提交

# Git的常用命令
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2019/2019-05-16-03.jpg" width="750"/></div>
&emsp;&emsp;
```
git init
    在本地新建一个repo,进入一个项目目录,执行git init,会初始化一个repo,并在当前文件夹下创建一个.git文件夹.
 
git clone
    获取一个url对应的远程Git repo, 创建一个local copy.
    一般的格式是git clone [url].
    clone下来的repo会以url最后一个斜线后面的名称命名,创建一个文件夹,如果想要指定特定的名称,可以git clone [url] newname指定.
 
git status
    查询repo的状态.
    git status -s: -s表示short, -s的输出标记会有两列,第一列是对staging区域而言,第二列是对working目录而言.
 
git log
    show commit history of a branch.
    git log --oneline --number: 每条log只显示一行,显示number条.
    git log --oneline --graph:可以图形化地表示出分支合并历史.
    git log branchname可以显示特定分支的log.
    git log --oneline branch1 ^branch2,可以查看在分支1,却不在分支2中的提交.^表示排除这个分支(Window下可能要给^branch2加上引号).
    git log --decorate会显示出tag信息.
    git log --author=[author name] 可以指定作者的提交历史.
    git log --since --before --until --after 根据提交时间筛选log.
    --no-merges可以将merge的commits排除在外.
    git log --grep 根据commit信息过滤log: git log --grep=keywords
    默认情况下, git log --grep --author是OR的关系,即满足一条即被返回,如果你想让它们是AND的关系,可以加上--all-match的option.
    git log -S: filter by introduced diff.
    比如: git log -SmethodName (注意S和后面的词之间没有等号分隔).
    git log -p: show patch introduced at each commit.
    每一个提交都是一个快照(snapshot),Git会把每次提交的diff计算出来,作为一个patch显示给你看.
    另一种方法是git show [SHA].
    git log --stat: show diffstat of changes introduced at each commit.
    同样是用来看改动的相对信息的,--stat比-p的输出更简单一些.
    
git add
    在提交之前,Git有一个暂存区(staging area),可以放入新添加的文件或者加入新的改动. commit时提交的改动是上一次加入到staging area中的改动,而不是我们disk上的改动.
    git add .  会递归地添加当前工作目录中的所有文件.
 
git diff
    不加参数的git diff:
    show diff of unstaged changes.
    此命令比较的是工作目录中当前文件和暂存区域快照之间的差异,也就是修改之后还没有暂存起来的变化内容.
 
    若要看已经暂存起来的文件和上次提交时的快照之间的差异,可以用:
    git diff --cached 命令.
    show diff of staged changes.
    (Git 1.6.1 及更高版本还允许使用 git diff --staged，效果是相同的).
 
    git diff HEAD
    show diff of all staged or unstated changes.
    也即比较woking directory和上次提交之间所有的改动.
 
    如果想看自从某个版本之后都改动了什么,可以用:
    git diff [version tag]
    跟log命令一样,diff也可以加上--stat参数来简化输出.
 
    git diff [branchA] [branchB]可以用来比较两个分支.
    它实际上会返回一个由A到B的patch,不是我们想要的结果.
    一般我们想要的结果是两个分支分开以后各自的改动都是什么,是由命令:
    git diff [branchA]…[branchB]给出的.
    实际上它是:git diff $(git merge-base [branchA] [branchB]) [branchB]的结果.
 
 
git commit
    提交已经被add进来的改动.
    git commit -m “the commit message"
    git commit -a 会先把所有已经track的文件的改动add进来,然后提交(有点像svn的一次提交,不用先暂存). 对于没有track的文件,还是需要git add一下.
    git commit --amend 增补提交. 会使用与当前提交节点相同的父节点进行一次新的提交,旧的提交将会被取消.
 
git reset
    undo changes and commits.
    这里的HEAD关键字指的是当前分支最末梢最新的一个提交.也就是版本库中该分支上的最新版本.
    git reset HEAD: unstage files from index and reset pointer to HEAD
    这个命令用来把不小心add进去的文件从staged状态取出来,可以单独针对某一个文件操作: git reset HEAD - - filename, 这个- - 也可以不加.
    git reset --soft
    move HEAD to specific commit reference, index and staging are untouched.
    git reset --hard
    unstage files AND undo any changes in the working directory since last commit.
    使用git reset —hard HEAD进行reset,即上次提交之后,所有staged的改动和工作目录的改动都会消失,还原到上次提交的状态.
    这里的HEAD可以被写成任何一次提交的SHA-1.
    不带soft和hard参数的git reset,实际上带的是默认参数mixed.
 
    总结:
    git reset --mixed id,是将git的HEAD变了(也就是提交记录变了),但文件并没有改变，(也就是working tree并没有改变). 取消了commit和add的内容.
    git reset --soft id. 实际上，是git reset –mixed id 后,又做了一次git add.即取消了commit的内容.
    git reset --hard id.是将git的HEAD变了,文件也变了.
    按改动范围排序如下:
    soft (commit) < mixed (commit + add) < hard (commit + add + local working)
 
git revert
    反转撤销提交.只要把出错的提交(commit)的名字(reference)作为参数传给命令就可以了.
    git revert HEAD: 撤销最近的一个提交.
    git revert会创建一个反向的新提交,可以通过参数-n来告诉Git先不要提交.
    
git rm
    git rm file: 从staging区移除文件,同时也移除出工作目录.
    git rm --cached: 从staging区移除文件,但留在工作目录中.
    git rm --cached从功能上等同于git reset HEAD,清除了缓存区,但不动工作目录树.
 
git clean
    git clean是从工作目录中移除没有track的文件.
    通常的参数是git clean -df:
    -d表示同时移除目录,-f表示force,因为在git的配置文件中, clean.requireForce=true,如果不加-f,clean将会拒绝执行.
 
git mv
    git rm - - cached orig; mv orig new; git add new
 
git stash
    把当前的改动压入一个栈.
    git stash将会把当前目录和index中的所有改动(但不包括未track的文件)压入一个栈,然后留给你一个clean的工作状态,即处于上一次最新提交处.
    git stash list会显示这个栈的list.
    git stash apply:取出stash中的上一个项目(stash@{0}),并且应用于当前的工作目录.
    也可以指定别的项目,比如git stash apply stash@{1}.
    如果你在应用stash中项目的同时想要删除它,可以用git stash pop
 
    删除stash中的项目:
    git stash drop: 删除上一个,也可指定参数删除指定的一个项目.
    git stash clear: 删除所有项目.
 
git branch
    git branch可以用来列出分支,创建分支和删除分支.
    git branch -v可以看见每一个分支的最后一次提交.
    git branch: 列出本地所有分支,当前分支会被星号标示出.
    git branch (branchname): 创建一个新的分支(当你用这种方式创建分支的时候,分支是基于你的上一次提交建立的). 
    git branch -d (branchname): 删除一个分支.
    删除remote的分支:
    git push (remote-name) :(branch-name): delete a remote branch.
    这个是因为完整的命令形式是:
    git push remote-name local-branch:remote-branch
    而这里local-branch的部分为空,就意味着删除了remote-branch
 
git checkout
    git checkout (branchname) 切换到一个分支.
    git checkout -b (branchname): 创建并切换到新的分支.
    这个命令是将git branch newbranch和git checkout newbranch合在一起的结果.
    checkout还有另一个作用:替换本地改动:
    git checkout --<filename>
    此命令会使用HEAD中的最新内容替换掉你的工作目录中的文件.已添加到暂存区的改动以及新文件都不会受到影响.
    注意:git checkout filename会删除该文件中所有没有暂存和提交的改动,这个操作是不可逆的.
 
git merge
    把一个分支merge进当前的分支.
    git merge [alias]/[branch]
    把远程分支merge到当前分支.
 
    如果出现冲突,需要手动修改,可以用git mergetool.
    解决冲突的时候可以用到git diff,解决完之后用git add添加,即表示冲突已经被resolved.
 
git tag
    tag a point in history as import.
    会在一个提交上建立永久性的书签,通常是发布一个release版本或者ship了什么东西之后加tag.
    比如: git tag v1.0
    git tag -a v1.0, -a参数会允许你添加一些信息,即make an annotated tag.
    当你运行git tag -a命令的时候,Git会打开一个编辑器让你输入tag信息.
    
    我们可以利用commit SHA来给一个过去的提交打tag:
    git tag -a v0.9 XXXX
 
    push的时候是不包含tag的,如果想包含,可以在push时加上--tags参数.
    fetch的时候,branch HEAD可以reach的tags是自动被fetch下来的, tags that aren’t reachable from branch heads will be skipped.如果想确保所有的tags都被包含进来,需要加上--tags选项.
 
git remote
    list, add and delete remote repository aliases.
    因为不需要每次都用完整的url,所以Git为每一个remote repo的url都建立一个别名,然后用git remote来管理这个list.
    git remote: 列出remote aliases.
    如果你clone一个project,Git会自动将原来的url添加进来,别名就叫做:origin.
    git remote -v:可以看见每一个别名对应的实际url.
    git remote add [alias] [url]: 添加一个新的remote repo.
    git remote rm [alias]: 删除一个存在的remote alias.
    git remote rename [old-alias] [new-alias]: 重命名.
    git remote set-url [alias] [url]:更新url. 可以加上—push和fetch参数,为同一个别名set不同的存取地址.
 
git fetch
    download new branches and data from a remote repository.
    可以git fetch [alias]取某一个远程repo,也可以git fetch --all取到全部repo
    fetch将会取到所有你本地没有的数据,所有取下来的分支可以被叫做remote branches,它们和本地分支一样(可以看diff,log等,也可以merge到其他分支),但是Git不允许你checkout到它们. 
 
git pull
    fetch from a remote repo and try to merge into the current branch.
    pull == fetch + merge FETCH_HEAD
    git pull会首先执行git fetch,然后执行git merge,把取来的分支的head merge到当前分支.这个merge操作会产生一个新的commit.    
    如果使用--rebase参数,它会执行git rebase来取代原来的git merge.
  
 
git rebase
    --rebase不会产生合并的提交,它会将本地的所有提交临时保存为补丁(patch),放在”.git/rebase”目录中,然后将当前分支更新到最新的分支尖端,最后把保存的补丁应用到分支上.
    rebase的过程中,也许会出现冲突,Git会停止rebase并让你解决冲突,在解决完冲突之后,用git add去更新这些内容,然后无需执行commit,只需要:
    git rebase --continue就会继续打余下的补丁.
    git rebase --abort将会终止rebase,当前分支将会回到rebase之前的状态.
 
git push
    push your new branches and data to a remote repository.
    git push [alias] [branch]
    将会把当前分支merge到alias上的[branch]分支.如果分支已经存在,将会更新,如果不存在,将会添加这个分支.
    如果有多个人向同一个remote repo push代码, Git会首先在你试图push的分支上运行git log,检查它的历史中是否能看到server上的branch现在的tip
    如果本地历史中不能看到server的tip,说明本地的代码不是最新的,Git会拒绝你的push,让你先fetch,merge,之后再push,这样就保证了所有人的改动都会被考虑进来.
 
git reflog
    git reflog是对reflog进行管理的命令,reflog是git用来记录引用变化的一种机制,比如记录分支的变化或者是HEAD引用的变化.
    当git reflog不指定引用的时候,默认列出HEAD的reflog.
    HEAD@{0}代表HEAD当前的值,HEAD@{3}代表HEAD在3次变化之前的值.
    git会将变化记录到HEAD对应的reflog文件中,其路径为.git/logs/HEAD, 分支的reflog文件都放在.git/logs/refs目录下的子目录中.
 
 
特殊符号:
    ^代表父提交,当一个提交有多个父提交时,可以通过在^后面跟上一个数字,表示第几个父提交: ^相当于^1.
    ~<n>相当于连续的<n>个^.
```

# 实践检验真理

## 仓库的创建

### create & clone
&emsp;&emsp;首先，在自己的GitHub（或者其他中心仓都OK）创建一个Repository（比如我创建了一个D02-GitPractice）；然后在本地磁盘找一个临时目录（比如我创建的D:\00-GitHub）。然后接下来就可以开始我们的Git实践之旅了。
&emsp;&emsp;要完成clone操作，首先要在GitHub仓库找到Repository的clone链接地址（通过【Clone or download】可以获取）；然后需要在本地的练习目录下，打开Git Bash窗口（右键即可找到对应的菜单项）；随后通过如下的clone命令完成仓库的clone，此时在本地磁盘就会发现多了一个D02-GitPractice的目录。
```bash

Administrator@Tang94 MINGW64 /d/00-GitHub
$ git clone git@github.com:ttfisher/D02-GitPractice.git
Cloning into 'D02-GitPractice'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.

Administrator@Tang94 MINGW64 /d/00-GitHub
$
```

### init
&emsp;&emsp;当然，实际使用时未必有GitHub环境或者中心仓环境，因为前面也说了Git是一个分布式的版本管理系统，所以自己完全也可以当做一个中心仓；此时init就可以充分发挥它的价值了，在任意目录下通过如下命令操作即可创建一个初始化好的本地仓库；这么做和从中心仓下载的区别，个人觉得最核心的是中心仓可以供团队作战进行资源汇总和交互。
```bash
Administrator@Tang94 MINGW64 /d/00-GitHub
$ git init myFirstGitRepo
Initialized empty Git repository in D:/00-GitHub/myFirstGitRepo/.git/

Administrator@Tang94 MINGW64 /d/00-GitHub
$
```

## 第一次提交
&emsp;&emsp;模拟第一次提交的场景：在本地仓库目录下新建一个文件 -> 为文件写入一些内容 -> 提交到本地仓 -> 推送到中心仓；本轮操作涉及到以下git命令：
- `git add` ： 将新增或修改添加到暂存区
- `git commit` ： 将暂存区的修改提交到本地仓
- `git push` ： 将本地仓的修改推送到远程仓
```bash
Administrator@Tang94 MINGW64 /d/00-GitHub
$ cd D02-GitPractice/

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ echo "Hello world." > 01.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ cat 01.txt
Hello world.

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git add .
warning: LF will be replaced by CRLF in 01.txt.
The file will have its original line endings in your working directory

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git commit  -m "This is my first commit."
[master 8b32a94] This is my first commit.
 1 file changed, 1 insertion(+)
 create mode 100644 01.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git  push origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 296 bytes | 296.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:ttfisher/D02-GitPractice.git
   1fe37c8..8b32a94  master -> master

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$
```
&emsp;&emsp;OK，推送完成后，我们到GitHub的D02-GitPractice，会发现文件列表多了一个刚刚我们在本地编辑的01.txt文件，具体展示是下面这样的，到这儿，我们就完成了第一次提交的操作，其实也蛮easy的。
```
01.txt        This is my first commit.        5 minutes ago
```

## 查看当前状态
&emsp;&emsp;通过`git status`命令，我们可以方便的查看本地仓库的状态，比如下面模拟的场景：
```bash

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ vi 01.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ echo "Hello world 2." > 02.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   01.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        02.txt

no changes added to commit (use "git add" and/or "git commit -a")

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git add 02.txt
warning: LF will be replaced by CRLF in 02.txt.
The file will have its original line endings in your working directory

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git commit -m "This is my second commit."
[master b936daa] This is my second commit.
 1 file changed, 1 insertion(+)
 create mode 100644 02.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   01.txt

no changes added to commit (use "git add" and/or "git commit -a")

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$
```

## 修改后的提交
&emsp;&emsp;其实和第一次提交没有太大区别，也是先要使用`git add`将改动增加到暂存区去，然后`git commit`时就会提交已经被add进来的改动，此时本地仓库就已经接收了这些改动，如果需要同步到中心仓则`git push`一下就Over了。因为这个过程也比较简单，这里就不模拟了。

## 分支的管理
&emsp;&emsp;几乎所有的版本控制系统都以某种形式支持分支，使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。等到分支线开发验证OK之后，再合入主线即可。以下模拟了从创建分支到合入主线的整个操作过程：
- `git branch` : 查看所有分支和当前所在分支
- `git branch xxx` ： 创建一个新分支
- `git checkout xxx` ： 切换到一个新分支
- `git merge xxx` ： 将xxx分支的修改合入当前分支（只针对本地仓生效，需要再push到中心仓才可以完成正式提交）
```bash
Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git branch
* master

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git branch dev

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git branch
  dev
* master

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git checkout dev
Switched to branch 'dev'

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (dev)
$ git branch
* dev
  master

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (dev)
$ echo "My dev branch" > 01.dev.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (dev)
$ git add 01.dev.txt
warning: LF will be replaced by CRLF in 01.dev.txt.
The file will have its original line endings in your working directory

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (dev)
$ git commit -m "First dev branch commit"
[dev 984d7ca] First dev branch commit
 1 file changed, 1 insertion(+)
 create mode 100644 01.dev.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (dev)
$ git push origin dev
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 352 bytes | 352.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote:
remote: Create a pull request for 'dev' on GitHub by visiting:
remote:      https://github.com/ttfisher/D02-GitPractice/pull/new/dev
remote:
To github.com:ttfisher/D02-GitPractice.git
 * [new branch]      dev -> dev

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (dev)
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git merge dev
Updating ca6eb1a..984d7ca
Fast-forward
 01.dev.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 01.dev.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git push origin master
Total 0 (delta 0), reused 0 (delta 0)
To github.com:ttfisher/D02-GitPractice.git
   ca6eb1a..984d7ca  master -> master

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$
```

## pull别人的提交
&emsp;&emsp;在这里，为了模拟pull操作，所以在本地又创建了一个Temp仓库来模拟提交了一次，主要涉及以下操作：
- 在本地Temp仓库新增或修改文件，并提交到远程仓库的master分支
- 在本地工作仓库使用`git pull`拉取远程仓库的修改
```bash
Administrator@Tang94 MINGW64 /d/99-Temp
$ cd D02-GitPractice/

Administrator@Tang94 MINGW64 /d/99-Temp/D02-GitPractice (master)
$ ll
total 4
-rw-r--r-- 1 Administrator 197121 15 7月  12 07:11 01.dev.txt
-rw-r--r-- 1 Administrator 197121 30 7月  12 07:11 01.txt
-rw-r--r-- 1 Administrator 197121 23 7月  12 07:11 02.txt
-rw-r--r-- 1 Administrator 197121 50 7月  12 07:11 README.md

Administrator@Tang94 MINGW64 /d/99-Temp/D02-GitPractice (master)
$ vi 01.txt

Administrator@Tang94 MINGW64 /d/99-Temp/D02-GitPractice (master)
$ echo Hahahah > t-01.txt

Administrator@Tang94 MINGW64 /d/99-Temp/D02-GitPractice (master)
$ git add .
warning: LF will be replaced by CRLF in t-01.txt.
The file will have its original line endings in your working directory

Administrator@Tang94 MINGW64 /d/99-Temp/D02-GitPractice (master)
$ git commit -m "Commit from temp folder."
[master 7b686fb] Commit from temp folder.
 2 files changed, 2 insertions(+)
 create mode 100644 t-01.txt

Administrator@Tang94 MINGW64 /d/99-Temp/D02-GitPractice (master)
$ git push origin master
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 356 bytes | 356.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:ttfisher/D02-GitPractice.git
   984d7ca..7b686fb  master -> master

Administrator@Tang94 MINGW64 /d/99-Temp/D02-GitPractice (master)
$ cd ../../00-GitHub/D02-GitPractice/

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ ll
total 4
-rw-r--r-- 1 Administrator 197121 15 7月  11 07:56 01.dev.txt
-rw-r--r-- 1 Administrator 197121 28 7月  11 07:22 01.txt
-rw-r--r-- 1 Administrator 197121 21 7月  11 07:40 02.txt
-rw-r--r-- 1 Administrator 197121 50 7月  11 07:06 README.md

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git pull origin
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 4 (delta 1), reused 4 (delta 1), pack-reused 0
Unpacking objects: 100% (4/4), done.
From github.com:ttfisher/D02-GitPractice
   984d7ca..7b686fb  master     -> origin/master
Updating 984d7ca..7b686fb
Fast-forward
 01.txt   | 1 +
 t-01.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 t-01.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ ll
total 5
-rw-r--r-- 1 Administrator 197121 15 7月  11 07:56 01.dev.txt
-rw-r--r-- 1 Administrator 197121 42 7月  12 07:15 01.txt
-rw-r--r-- 1 Administrator 197121 21 7月  11 07:40 02.txt
-rw-r--r-- 1 Administrator 197121 50 7月  11 07:06 README.md
-rw-r--r-- 1 Administrator 197121  9 7月  12 07:15 t-01.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$
```

## 怎么打tag？
&emsp;&emsp;通过`git tag`的不同使用，可以完成打标签的操作，比如按照下面的流程进行操作之后，就可以在远程仓库看到这个1.0.0的tag，所谓的tag就是一个标记。
&emsp;&emsp;这么一看，tag和branch是有些类似的，不过二者还是有本质区别的：tag对应某次commit,是一个点，是不可移动的；branch对应一系列commit，是很多点连成的一根线，有一个HEAD指针，是可以依靠HEAD指针移动的。
```bash
Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master|MERGING)
$ git tag

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master|MERGING)
$ git tag -a 1.0.0 -m "This is tag 1.0.0"

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master|MERGING)
$ git tag
1.0.0

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master|MERGING)
$ git push origin --tags
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 434 bytes | 434.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:ttfisher/D02-GitPractice.git
 * [new tag]         1.0.0 -> 1.0.0

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master|MERGING)
$
```
&emsp;&emsp;以下列举出tag常用的方式，仅供参考：
```bash
#// 打印当前本地所有标签
git tag

# 带过滤条件的tag列表
git tag -l t-*.1

# 查看某个标签的状态
git checkout 1.0.0

# 本地仓库创建轻量标签
git tag 1.0.0-light

# 本地仓库创建带备注标签【推荐使用】
git tag -a 1.0.0 -m "This is comment"

# 本地仓库对特定commit版本SHA创建标签
git tag -a 1.0.0 0x2c51f -m "This is comment"

# 本地仓库删除标签
git tag -d 1.0.0

# 将本地所有标签发布到远程仓库
git push origin --tags

# 将本地某个标签发布到远程仓库
git push origin 1.0.0

# 删除远程仓库的标签（常用第一个，后面一个是老版本git使用的）
git push origin --delete 1.0.0
git push origin :refs/tags/1.0.0
```

# 实用小技巧

## 三个区的区别
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2019/2019-07-11-01.png" width="750"/></div>
- 工作区：就是你在电脑里能看到的目录。
- 暂存区：英文叫stage, 或index。一般存放在 ".git目录下" 下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
- 版本库：工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

## git commit提交哪些文件怎么控制？
&emsp;&emsp;`git commit`命令，提交的是已经加入暂存区(staging area)的内容，比如你本地修改或者新增了一个文件，但是没有add，那么commit时就不会提交这个文件。
&emsp;&emsp;实际提交的流程就如同上图所示，图中左侧为工作区（对应我们本地目录），右侧为版本库（对应Git仓库）。在版本库中标记为 "index" 的区域是暂存区（stage, index），标记为 "master" 的是 master 分支所代表的目录树。
&emsp;&emsp;git提交时也是可以实现工作区和暂存区的文件一起提交的，比如下面模拟的操作我们就对两个区域的文件同时进行了commit，核心的命令是`git commit -o work.area.file stage.area.file -m 'Commit files in two areas'`：
```bash
Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   01.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   02.txt

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git commit -o 01.txt 02.txt -m 'Commit files in two areas'
warning: LF will be replaced by CRLF in 02.txt.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in 01.txt.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in 02.txt.
The file will have its original line endings in your working directory
[master ca6eb1a] Commit files in two areas
 2 files changed, 2 insertions(+)

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$ git push origin master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 372 bytes | 372.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To github.com:ttfisher/D02-GitPractice.git
   b936daa..ca6eb1a  master -> master

Administrator@Tang94 MINGW64 /d/00-GitHub/D02-GitPractice (master)
$
```