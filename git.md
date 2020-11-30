# git用法

## 创建版本库

> 1. 将文件夹变为可以管理的仓库
>
> `$ git init`
>
> 2. 把文件添到版本库
>
> `$ git add readme.txt`
>
> 如过出现`warning: LF will be replaced by CRLF in readme.txt.`,可以`git config --global core.autocrlf false`
>
> 3. 将文件添加到仓库,`-m`后面输入的是本次提交的说明
>
> `$ git commit -m "wrote a readme file"`
>
> `commit`可以一次提交很多文件，所以可以多次`add`不同的文件
>
> ```
> $ git add file1.txt
> $ git add file2.txt file3.txt
> $ git commit -m "add 3 files."
> ```

## 时空机穿梭

我们已经成功地添加并提交了一个readme.txt文件，现在，是时候继续工作了，于是，我们继续修改readme.txt文件，改成如下内容：

```
Git is a distributed version control system.
Git is free software.
```

现在，运行`git status`命令看看结果：

```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

`git status`命令可以让我们时刻掌握仓库当前的状态，上面的命令输出告诉我们，`readme.txt`被修改过了，但还没有准备提交的修改。

虽然Git告诉我们`readme.txt`被修改了，但如果能看看具体修改了什么内容，自然是很好的。比如你休假两周从国外回来，第一天上班时，已经记不清上次怎么修改的`readme.txt`，所以，需要用`git diff`这个命令看看：

```
$ git diff readme.txt 
diff --git a/readme.txt b/readme.txt
index 46d49bf..9247db6 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.
```

`git diff`顾名思义就是查看difference，显示的格式正是Unix通用的diff格式，可以从上面的命令输出看到，我们在第一行添加了一个`distributed`单词。

知道了对`readme.txt`作了什么修改后，再把它提交到仓库就放心多了，提交修改和提交新文件是一样的两步，第一步是`git add`：

```
$ git add readme.txt
```

同样没有任何输出。在执行第二步`git commit`之前，我们再运行`git status`看看当前仓库的状态：

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   readme.txt
```

`git status`告诉我们，将要被提交的修改包括`readme.txt`，下一步，就可以放心地提交了：

```
$ git commit -m "add distributed"
[master e475afc] add distributed
 1 file changed, 1 insertion(+), 1 deletion(-)
```

提交后，我们再用`git status`命令看看仓库的当前状态：

```
$ git status
On branch master
nothing to commit, working tree clean
```

Git告诉我们当前没有需要提交的修改，而且，工作目录是干净（working tree clean）的。

### 版本回退

`git log`命令显示从最近到最远的提交日志

如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数

在Git中，用`HEAD`表示当前版本，也就是最新的提交`1094adb...`（注意我的提交ID和你的肯定不一样），上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

将当前版本回退到上一个版本：

```
$ git reset --hard HEAD^
```

想要回到修改前的版本（未来的版本）:

* 如果控制面板还没有关掉，查看未来版本的commit id,于是就可以指定回到未来的某个版本：

```
$ git reset --hard 1094a
```

* 已经关掉了控制面板，想找到未来版本的commit id.

  `git reflog`命令记录了每一次命令

  ```
  $ git reflog
  1a63adb (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
  822aa32 HEAD@{1}: commit: append GPL
  1a63adb (HEAD -> master) HEAD@{2}: commit: add distributed
  70ef1c2 HEAD@{3}: commit (initial): worte a readme.txt
  $ git reset --hard 822a
  ```

```
如果已经有A -> B -> C，想回到B：

方法一：reset到B，丢失C：

A -> B

方法二：再提交一个revert反向修改，变成B：

A -> B -> C -> B

C还在，但是两个B是重复的

看你的需求，也许C就是瞎提交错了（比如把密码提交上去了），必须reset

如果C就是修改，现在又要改回来，将来可能再改成C，那你就revert
最重要的一点：revert 是回滚某个 commit ，不是回滚“到”某个
```



### 工作区和暂存区

**工作区（Working Directory）**

就是你在电脑里能看到的目录，比如我的`git_test`文件夹就是一个工作区

**版本库（Repository）**

工作区有一个隐藏目录`.git`，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

```
暂存区的文件目录和最近一次git add之后工作区的文件目录是一致的，因此当你修改完文件使用git diff你可以看到工作区和暂存区的差异（此时暂存区是上一次add之后的目录，而工作区是你刚修改完的样子），使用git add加入新的修改，再git diff就不会有输出，因为暂存区被更新为最新的工作区目录。 git diff --cached同理，只是比较的是暂存区和分支里的内容差异（分支保持上一次git commit的目录）。
```

```
Git管理的文件分为：工作区，版本库，版本库又分为暂存区stage和暂存区分支master(仓库)

工作区>>>>暂存区>>>>仓库

git add把文件从工作区>>>>暂存区，git commit把文件从暂存区>>>>仓库，

git diff查看工作区和暂存区差异，

git diff --cached查看暂存区和仓库差异，

git diff HEAD 查看工作区和仓库的差异，

git add的反向命令git checkout，撤销工作区修改，即把暂存区最新版本转移到工作区，

git commit的反向命令git reset HEAD，就是把仓库最新版本转移到暂存区。

git reset --hard HEAD 命令 会将工作区和暂存区恢复成HEAD
```



### 管理修改

Git跟踪并管理的是修改，而非文件。

比如你新增了一行，这就是一个修改，删除了一行，也是一个修改，更改了某些字符，也是一个修改，删了一些又加了一些，也是一个修改，甚至创建一个新文件，也算一个修改。

每次修改，如果不用`git add`到暂存区，那就不会加入到`commit`中。



### 撤销修改

`git checkout -- file`可以丢弃工作区的修改：

```
$ git checkout -- readme.txt
```

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。



如果已经提交到缓存区，还没有commit.

用命令`git reset HEAD <file>`可以把暂存区的修改撤销掉，重新放回工作区：

再使用`git checkout --readme.txt`



场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考[版本回退](#版本回退)一节，不过前提是没有推送到远程库。



### 删除文件

1.如果你用的`rm`删除文件，那就相当于只删除了工作区的文件，如果想要恢复，直接用`git checkout -- <file>`就可以 2.如果你用的是git `rm`删除文件，那就相当于不仅删除了文件，而且还添加到了暂存区，需要先`git reset HEAD <file>`，然后再`git checkout -- <file> `3.如果你想彻底把版本库的删除掉，先`git rm`，再`git commit`就ok了。



一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用`rm`命令删了：

```
$ rm test.txt
```

这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，`git status`命令会立刻告诉你哪些文件被删除了



现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令`git rm`删掉，并且`git commit`：

```
$ git rm test.txt
rm 'test.txt'

$ git commit -m "remove test.txt"
[master d46f35e] remove test.txt
 1 file changed, 1 deletion(-)
 delete mode 100644 test.txt
```

现在，文件就从版本库中被删除了。

```
先手动删除文件，然后使用git rm <file>和git add<file>效果是一样的。
```

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：

```
$ git checkout -- test.txt
```

`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。



## 远程仓库

### 添加远程仓库

现在的情景是，你已经在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库既可以作为备份，又可以让其他人通过该仓库来协作，真是一举多得。

首先，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库：

![github-create-repo-1](https://www.liaoxuefeng.com/files/attachments/919021631860000/0)

在Repository name填入`learngit`，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库：

![github-create-repo-2](https://www.liaoxuefeng.com/files/attachments/919021652277920/0)

目前，在GitHub上的这个`learngit`仓库还是空的，GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。

现在，我们根据GitHub的提示，在本地的`learngit`仓库下运行命令：

```
$ git remote add origin git@github.com:michaelliao/learngit.git
```

请千万注意，把上面的`michaelliao`替换成你自己的GitHub账户名，否则，你在本地关联的就是我的远程库，关联没有问题，但是你以后推送是推不上去的，因为你的SSH Key公钥不在我的账户列表中。

添加后，远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是`origin`这个名字一看就知道是远程库。

下一步，就可以把本地库的所有内容推送到远程库上：

```
$ git push -u origin master
Counting objects: 20, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (20/20), 1.64 KiB | 560.00 KiB/s, done.
Total 20 (delta 5), reused 0 (delta 0)
remote: Resolving deltas: 100% (5/5), done.
To github.com:michaelliao/learngit.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

推送成功后，可以立刻在GitHub页面中看到远程库的内容已经和本地一模一样：

![github-repo](https://www.liaoxuefeng.com/files/attachments/919021675995552/0)

从现在起，只要本地作了提交，就可以通过命令：

```
$ git push origin master
```

把本地`master`分支的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库！

**SSH警告**

当你第一次使用Git的`clone`或者`push`命令连接GitHub时，会得到一个警告：

```
The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
RSA key fingerprint is xx.xx.xx.xx.xx.
Are you sure you want to continue connecting (yes/no)?
```

这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入`yes`回车即可。

Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：

```
Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
```

这个警告只会出现一次，后面的操作就不会有任何警告了。

如果你实在担心有人冒充GitHub服务器，输入`yes`前可以对照[GitHub的RSA Key的指纹信息](https://help.github.com/articles/what-are-github-s-ssh-key-fingerprints/)是否与SSH连接给出的一致。

**小结**

要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；

关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；

分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，也就是有没有联网都可以正常工作，而SVN在没有联网的时候是拒绝干活的！当有网络的时候，再把本地提交推送一下就完成了同步，真是太方便了！

### 从远程库克隆

上次我们讲了先有本地库，后有远程库的时候，如何关联远程库。

现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。

首先，登陆GitHub，创建一个新的仓库，名字叫`gitskills`：

![github-init-repo](https://www.liaoxuefeng.com/files/attachments/919021808263616/0)

我们勾选`Initialize this repository with a README`，这样GitHub会自动为我们创建一个`README.md`文件。创建完毕后，可以看到`README.md`文件：

![github-init-repo-2](https://www.liaoxuefeng.com/files/attachments/919021836828288/0)

现在，远程库已经准备好了，下一步是用命令`git clone`克隆一个本地库：

```
$ git clone git@github.com:michaelliao/gitskills.git
Cloning into 'gitskills'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 3
Receiving objects: 100% (3/3), done.
```

注意把Git库的地址换成你自己的，然后进入`gitskills`目录看看，已经有`README.md`文件了：

```
$ cd gitskills
$ ls
README.md
```

如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了。

你也许还注意到，GitHub给出的地址不止一个，还可以用`https://github.com/michaelliao/gitskills.git`这样的地址。实际上，Git支持多种协议，默认的`git://`使用`ssh`，但也可以使用`https`等其他协议。

使用`https`除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用`ssh`协议而只能用`https`。

**小结**

要克隆一个仓库，首先必须知道仓库的地址，然后使用`git clone`命令克隆。

Git支持多种协议，包括`https`，但`ssh`协议速度最快。



### SSH问题

本地git bash 使用git clone git@github.com:***.git方式下载`github`代码至本地时需要依赖`ssh key`，遇到权限不足问题时一般都是SSH key失效或者SSH key不存在，重新创建SSH key一般就可以解决问题；

**步骤一、检查本地`ssh key`是否存在**

  1、windows下 开始 -- 搜索框输入 git bash，打开git bash窗口；

  2、git base窗口中输入指令`ls ~/.ssh/` 来检查`ssh key`是否存在；

  3、如果key不存在则按照步骤二重新生成，`ssh key`已存在则跳过步骤二，执行步骤三；

**步骤二、生成`ssh key`**

  1、继续步骤一的git bash窗口执行指令：

​      `ssh-keygen -t rsa -b 2048 -C "你自己的邮箱地址"`

​      修改邮箱地址为你自己的邮箱地址，注意此处邮箱地址前后的双引号为英文格式双引号；

  2、指令执行后页面提示：

​      ![img](https://img-blog.csdn.net/20180804171826436?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N4ZzAyMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

      ```
Generating public/private rsa key pair.
      Enter file in which to save the key (/c/Users/***/.ssh/id_rsa):

​     ***表示你自己的当前登录用户名，不做修改直接回车，会将生成的rsa文件保存为默认名称

​     再次回车提示：

​     Enter passphrase (empty for no passphrase): 
​     Enter same passphrase again: 
​     提示设置提交/l拉取代码到`Github`时需要的密码及确认密码；

​     设置密码后再次回车提示Your identification has been saved in.... 即表示ssh key生成成功；
      ```



**步骤三、添加`sshkey`至`ssh-agent`**

  1、执行`eval “$(ssh-agent -s)”`确认`ssh-agent`处于开启状态，打印`pid... `表示启用中；

  2、执行指令`ssh-add ~/.ssh/id_rsa` 添加`ssh key`至`ssh agent`，此步会要求输入步骤二设置的密码；

​     需要注意的是此处可能报错：Could not open a connection to your authentication agent

```
执行ssh-add时出现Could not open a connection to your authentication agent

在执行 ssh-add ~/.ssh/id_ras 时发生此错，

执行如下命令　ssh-agent bash
然后再执行 ssh-add ~/.ssh/id_rsa 即可。
```



**步骤四、添加`ssh key`至`github`**

   1、登录https://github.com/，在页面右上角自己头像右边箭头处右击，弹框中进入setting功能；

​      ![img](https://img-blog.csdn.net/20180804172820648?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N4ZzAyMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   2、setting界面右边菜单选择SSH and GPG keys，选择新建SSH keys，

![img](https://img-blog.csdn.net/20180804172926782?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N4ZzAyMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  ![img](https://img-blog.csdn.net/20180804173209804?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N4ZzAyMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

保存即可；

**步骤五：git clone下载代码**

  步骤结束，此时再尝试本地使用git clone方式下载代码即可；







