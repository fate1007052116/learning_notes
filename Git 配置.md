# Git 配置

```sh
git config --global user.name "luo"
git config --global user.email "1007052116@qq.com"
```

当前目录下有隐藏文件 .git 说明 当前文件夹是git所管理的

```she
git init # 初始化当前文件夹为git所管理的文件夹
git add test.txt # 将此文件添加为git所管理的文件
git commit -m "提交的描述"  # 提交当前文件夹下git所管理的文件
git status
git add     # 将文件添加到git的缓存中，并且需要git commit 才能将该缓存中的文件放入到真正的git 仓库中
						# 并将git 仓库中已有的同名同目录文件做增量的保存（文件快照）
						# 文件提交时，必须先git add 再 git commit 才能提交
```

### 1. git 存储流程

代码工作区-->执行git add -->暂存区（临时存储，本次的提交清单）-->执行git commit -m "描述" -->本地库（历史版本）

工作区、暂存区、本地仓库，逻辑上都是本地计算机。

* 当我们创建一个新文件时，文件位于工作区，出于已修改(modify)的状态，表明文件已经进行了修改，但是还没有保存，通过git add 将其添加到暂存区，文件时已暂存(staged) 状态，表明把已修改的文件放到下次要提交时要保存的清单中；通过git commit -m "" 将文件放入本地仓库，文件为已提交（commit）状态，表明该文件已经安全地保存在本地数据库中，这是，就的版本会生成一个新的快照，新提交的代码在新快照的基础上做增量添加

```shell
git diff test.txt #查看未提交版本和git库中的版本有哪些区别
git log #查看日志
git log --pretty=oneline # 每次提交在日志中只显示为一行 ()为最新版本
git reflog # 引用日志，最新版本 HEAD@{0}
git reset --hard (git reflog 中开头的版本号)
```

```shell
# 回滚示例,本地工作区的文件也随之会滚到指定版本，
luo@luodeMacBook-Pro myFirstGitProject % git reflog
252030a (HEAD -> master) HEAD@{0}: commit: 改了
04d3f67 HEAD@{1}: commit: 第三次确认提交
d8ab018 HEAD@{2}: commit (initial): 第二次提交
luo@luodeMacBook-Pro myFirstGitProject % git reset --hard d8ab018
HEAD is now at d8ab018 第二次提交
# 本地工作区回滚前的版本则被保存，从HEAD@{0} 变为 HEAD@{1}
luo@luodeMacBook-Pro myFirstGitProject % git reflog
d8ab018 (HEAD -> master) HEAD@{0}: reset: moving to d8ab018
252030a HEAD@{1}: commit: 改了
04d3f67 HEAD@{2}: commit: 第三次确认提交
d8ab018 (HEAD -> master) HEAD@{3}: commit (initial): 第二次提交
```

### 删除文件

```shell
rm test.txt #删除工作区文件
git add test.txt
git commit -m "删除文件"
```

### 恢复文件

```shell
# 方式一
git reflog # 找到最新版本的id
git reset --hard 252030a #再做恢复
#方式二
git checkout -- test.txt # 会自动同步最新的文件到本地(需要在未提交之前)
```

### 分支管理操作

```shell
git branch b1 # 分支可以叫除master之外的任一名字
git branch -v # 查看主干和分支的版本  *代表当前正在使用的分支
git checkout b1 # 根据分支的名称，切换到该分支
```

### 分支的合并

```shell
# 首先需要切换到需要合并的分支 (举例，切换到master进行合并分支)
git checkout master
# 将特定分支合并到当前分支
git merge b1 # 将b1分支合并到当前分支（当前分支为master）
# 合并完成之后的分支就可以销毁了,不要删除已选定的分支（删除自己）
git branch -d b1
```

### SSH-key

```shell
luo@luodeMacBook-Pro myFirstGitProject % ssh-keygen -t rsa -C "1007052116@qq.com" # -t rsa 指 使用RSA加密协议
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/luo/.ssh/id_rsa):
Enter passphrase (empty for no passphrase): #输入断点信息，（我没有输入）
Enter same passphrase again:
Your identification has been saved in /Users/luo/.ssh/id_rsa.
Your public key has been saved in /Users/luo/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:uH/0rZz4A5UTaQyO8ZT9Yvas04zoelTPD+dgU9jlkRU 1007052116@qq.com
The key's randomart image is:
+---[RSA 2048]----+
|        . o= . E=|
|         *. *  o.|
|        . o. + +o|
|       .    O o +|
|      . S  = B . |
|       .  +   X .|
|      .  o + B B |
|       .  +o*.+ o|
|        o=o.=+   |
+----[SHA256]-----+

#验证SSH-key 是否配置成功
luo@luodeMacBook-Pro .ssh % ssh -T git@git.oschina.net
The authenticity of host 'git.oschina.net (212.64.62.183)' can't be established.
ECDSA key fingerprint is SHA256:FQGC9Kn/eye1W8icdBgrQp+KkGYoFgbVr17bmjey0Wc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'git.oschina.net,212.64.62.183' (ECDSA) to the list of known hosts.
Hi 罗俊! You've successfully authenticated, but GITEE.COM does not provide shell access.

# 查看公共密钥，去码云上设置公钥
luo@luodeMacBook-Pro conf % cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDnnk1u0DfeoY5mwkEEN8+Bdz5Il2a2P2Cnca9DaPfexdjI9lC7fXkt4oUfdGm4JVzGqidxDA8gRMK2dMK5X5GHWajwlgQsTQHYbZyT9yTslmdHChm5lsDjIlu30HS8iThpLAwcoKDt+aV/ao20Yn1j1+5BLOmCDjTyW0VI7GVYMgsCBYC3XcRA3q1IKcxl8RLwJnNUCvkLIH/xvoqlTzbdIjXXiYSRJo0clohL2XycVsEDa/hbFISu8GG8y5K12Cpxi3UuIBPM5lRadqtWs4ZJ0hcpasDe93h0iQDmZplt2aZqqvWCCXpdED4FBBLVmtKzbIi/QNUHsfk5HcFwN3S5 1007052116@qq.com

# 查看私钥key
luo@luodeMacBook-Pro conf % cat ~/.ssh/id_rsa
```

### 创建本地仓库，从无到有，从0创建

```shell
luo@luodeMacBook-Pro git % pwd
/Volumes/OS/javaEE/git
luo@luodeMacBook-Pro git % mkdir mySecondRepos
luo@luodeMacBook-Pro git % cd mySecondRepos
luo@luodeMacBook-Pro mySecondRepos %
# 至此只是创建了目录，还没有进行初始化
# 克隆远程仓库到本地
luo@luodeMacBook-Pro mySecondRepos % git clone https://gitee.com/luo_jun99/myFirstRepo.git
Cloning into 'myFirstRepo'...
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 7 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (7/7), done.
luo@luodeMacBook-Pro mySecondRepos % pwd
/Volumes/OS/javaEE/git/mySecondRepos
```

```shell
git add ./ Test.java # 放到当前目录下
```

### 推送到git仓库

```shell
luo@luodeMacBook-Pro myFirstRepo % git push https://gitee.com/luo_jun99/myFirstRepo.git master #master 分支 可以省略
Username for 'https://gitee.com': 1007052116@qq.com
Password for 'https://1007052116@qq.com@gitee.com':
Counting objects: 6, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 554 bytes | 554.00 KiB/s, done.
Total 6 (delta 2), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-5.0]
To https://gitee.com/luo_jun99/myFirstRepo.git
   d7d619f..7692e30  master -> master
luo@luodeMacBook-Pro myFirstRepo %
```

### 从git仓库拉取到本地，在本地现有文件基础上做增量操作，不会覆盖

![Snip20200909_1](/Users/luo/Documents/开发笔记/images/Snip20200909_1.png)

```shell
luo@luodeMacBook-Pro myFirstRepo % git pull https://gitee.com/luo_jun99/myFirstRepo.git
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://gitee.com/luo_jun99/myFirstRepo
 * branch            HEAD       -> FETCH_HEAD
Updating 7692e30..b722eed
Fast-forward
 application.yml | 2 ++
 1 file changed, 2 insertions(+)
 create mode 100644 application.yml
```

### 其他命令

```shell
luo@luodeMacBook-Pro myFirstRepo % git remote -v
origin	https://gitee.com/luo_jun99/myFirstRepo.git (fetch) #fetch 下载
origin	https://gitee.com/luo_jun99/myFirstRepo.git (push)  #push 上传
# orgin为别名
# 为远程仓库添加别名，别名为 myFirstRepo ，但是不推荐，容易记混淆
git remote add myFirstRepo https://gitee.com/luo_jun99/myFirstRepo.get
# 删除别名
git remote remove myFirstRepo
```

### 解决包冲突，冲突之后git会创建一个临时分支来缓存冲突，手动解决冲突之后，需要将临时分支和主分支进行合并

```shell
# push之前需要先pull
luo@luodeMacBook-Pro myFirstRepo % git pull https://gitee.com/luo_jun99/myFirstRepo.git
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://gitee.com/luo_jun99/myFirstRepo
 * branch            HEAD       -> FETCH_HEAD
Auto-merging application.yml																# git已经将冲突的文件合并，等待修改
CONFLICT (content): Merge conflict in application.yml				# 从远程仓库下载的时候发现冲突
Automatic merge failed; fix conflicts and then commit the result.
luo@luodeMacBook-Pro myFirstRepo % tail application.yml			# 查看冲突
server:
<<<<<<< HEAD			# <<< 代表时本地
    port: 8080
=======						# === 代表远程仓库
    port: 9090
>>>>>>> 0f8d3cdc4f92590fed33e41cb9ac22a5123f126e
luo@luodeMacBook-Pro myFirstRepo % git reflog							# 从远程仓库下载之后还没有提交本地仓库
9ae96ad (HEAD -> master) HEAD@{0}: commit: 修改tomcat
b722eed HEAD@{1}: pull https://gitee.com/luo_jun99/myFirstRepo.git: Fast-forward
7692e30 HEAD@{2}: commit: 添加了main方法
6a2fc30 HEAD@{3}: commit: test.java commit 1
d7d619f (origin/master, origin/HEAD) HEAD@{4}: clone: from https://gitee.com/luo_jun99/myFirstRepo.git   # 本地文件和远程文件合并之后的日志
# 手动解决冲突
luo@luodeMacBook-Pro myFirstRepo % vi application.yml
luo@luodeMacBook-Pro myFirstRepo % cat application.yml
server:
    port: 8080		#解决冲突之后
luo@luodeMacBook-Pro myFirstRepo % git add application.yml		# 在本地仓库提交
luo@luodeMacBook-Pro myFirstRepo % git commit -m "解决了本地和远程仓库之间的端口冲突"
[master b776455] 解决了本地和远程仓库之间的端口冲突
# 解决冲突之后才能再次提交远程仓库
luo@luodeMacBook-Pro myFirstRepo % git push https://gitee.com/luo_jun99/myFirstRepo.git
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 656 bytes | 656.00 KiB/s, done.
Total 5 (delta 1), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-5.0]
To https://gitee.com/luo_jun99/myFirstRepo.git
   0f8d3cd..b776455  master -> master
# 查看合并之后的日志
luo@luodeMacBook-Pro myFirstRepo % git reflog
b776455 (HEAD -> master) HEAD@{0}: commit (merge): 解决了本地和远程仓库之间的端
口冲突		# 有merge 代表的是合并
9ae96ad HEAD@{1}: commit: 修改tomcat
b722eed HEAD@{2}: pull https://gitee.com/luo_jun99/myFirstRepo.git: Fast-forward
7692e30 HEAD@{3}: commit: 添加了main方法
6a2fc30 HEAD@{4}: commit: test.java commit 1
d7d619f (origin/master, origin/HEAD) HEAD@{5}: clone: from https://gitee.com/luo_jun99/myFirstRepo.git
```

### 新增分支并推送到git仓库

```shell
luo@luodeMacBook-Pro myFirstRepo % git branch b1
luo@luodeMacBook-Pro myFirstRepo % git checkout b1
Switched to branch 'b1'
luo@luodeMacBook-Pro myFirstRepo %  git push https://gitee.com/luo_jun99/myFirstRepo.git
Total 0 (delta 0), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-5.0]
remote: Create a pull request for 'b1' on Gitee by visiting:
remote:     https://gitee.com/luo_jun99/myFirstRepo/pull/new/luo_jun99:b1...luo_jun99:master
To https://gitee.com/luo_jun99/myFirstRepo.git
 * [new branch]      b1 -> b1
```

### 修改b1分支，并上传到远程仓库

```shell
luo@luodeMacBook-Pro myFirstRepo % echo " public class T2{}" >>t2.java
luo@luodeMacBook-Pro myFirstRepo % git add t2.java
luo@luodeMacBook-Pro myFirstRepo % git commit -m "新建t2.java 到b2分支下"
[b1 c8e6c71] 新建t2.java 到b2分支下
 1 file changed, 1 insertion(+)
 create mode 100644 t2.java
luo@luodeMacBook-Pro myFirstRepo % git push https://gitee.com/luo_jun99/myFirstRepo.git
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 307 bytes | 307.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-5.0]
To https://gitee.com/luo_jun99/myFirstRepo.git
   b776455..c8e6c71  b1 -> b1
luo@luodeMacBook-Pro myFirstRepo %
# 此时远程仓库只有b1分支的代码被改动，master分支的代码没有改变
```

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200626095935803.png" alt="image-20200626095935803" style="zoom:50%;" />

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200626095957328.png" alt="image-20200626095957328" style="zoom:50%;" />

### 合并分支

```shell
luo@luodeMacBook-Pro myFirstRepo % git checkout master #先切换到主分支
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)
luo@luodeMacBook-Pro myFirstRepo % git merge b1 	#再合并分支
Updating b776455..c8e6c71
Fast-forward
 t2.java | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 t2.java
# 将已合并的主分支提交到远程仓库
luo@luodeMacBook-Pro myFirstRepo % git push https://gitee.com/luo_jun99/myFirstRepo.git
Total 0 (delta 0), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-5.0]
To https://gitee.com/luo_jun99/myFirstRepo.git
   b776455..c8e6c71  master -> master
luo@luodeMacBook-Pro myFirstRepo %
#此时发现主分支已经有了b1分支的内容
```

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200626100650383.png" alt="image-20200626100650383" style="zoom:50%;" />

### 删除本地分支，再推送到仓库（不能通过删除本地分支来删除远程分支，远程分支必须在远程仓库删除）

```shell
luo@luodeMacBook-Pro myFirstRepo % git branch -d b1
Deleted branch b1 (was c8e6c71).
luo@luodeMacBook-Pro myFirstRepo % git branch -v
* master c8e6c71 [ahead 7] 新建t2.java 到b2分支下
# 推送
luo@luodeMacBook-Pro myFirstRepo % git push https://gitee.com/luo_jun99/myFirstRepo.git
Everything up-to-date
# 本次操作删除的是本地分支，远程仓库的分支不会被删除
```

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200626101045441.png" alt="image-20200626101045441" style="zoom:50%;" />

## IDEA中git的文件颜色对应状态

#### git默认不会管理空文件夹

* 红色：没有使用git add 命令，是全新的文件
* 绿色：已经使用git add 命令，但是仓库里没有这个文件，未提交状态
* 蓝色：仓库里已经有这个文件，但是没有做提交
* 一旦使用了git add ，之后每修改一个文件，IDEA都会自动为这个文件添加缓存 git add



## IDEA 中不能pull远程仓库中的内容与本地git中内容合并的时候

```shell
# 忽略未同步的历史，忽略本地版本，做强合并
# 这里的 origin 使用了别名 https 
luo@luodeMacBook-Pro springfoxDemo % git pull origin master --allow-unrelated-histories
From https://gitee.com/luo_jun99/myFirstRepo
 * branch            master     -> FETCH_HEAD
Merge made by the 'recursive' strategy.   # 此时已经合并完成，并创建了一个临时的文件，并且
 .gitee/ISSUE_TEMPLATE.zh-CN.md        | 13 ++++++++++++
 .gitee/PULL_REQUEST_TEMPLATE.zh-CN.md | 15 ++++++++++++++
 README.en.md                          | 36 ++++++++++++++++++++++++++++++++
 README.md                             | 39 +++++++++++++++++++++++++++++++++++
 Test.java                             |  6 ++++++
 application.yml                       |  2 ++
 t2.java                               |  1 +
 7 files changed, 112 insertions(+)
 create mode 100644 .gitee/ISSUE_TEMPLATE.zh-CN.md
 create mode 100644 .gitee/PULL_REQUEST_TEMPLATE.zh-CN.md
 create mode 100644 README.en.md
 create mode 100644 README.md
 create mode 100644 Test.java
 create mode 100644 application.yml
 create mode 100644 t2.java
```

### 将当前文件夹下git的所管理的内容上传到服务器

```shell
luo@luodeMacBook-Pro springfoxDemo % git push origin master -f
Counting objects: 38, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (28/28), done.
Writing objects: 100% (38/38), 57.27 KiB | 7.16 MiB/s, done.
Total 38 (delta 1), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-5.0]
To https://gitee.com/luo_jun99/myFirstRepo.git
   c8e6c71..468572b  master -> master
```

### 将特定文件从已修改状态回滚到某一记录

```shell
git checkout 51073e3 iwmake-common/iwmake-common-core/src/main/java/com/iwmake/common/core/utils/TgmkbitUtils.java
```

