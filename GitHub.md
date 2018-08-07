## git

1. git是什么：是一个开源的分布式版本控制系统，可以对项目进行版本管理。早期是linux之父来管理系统源代码
2. 常见的源代码管理工具
   1. git：分布式版本控制系统
   2. svn：集中式版本控制系统
3. 什么是版本控制系统
   - 用来控制源代码的版本，软件升级后，代码也会变化，代码也需要有一个版本来规范（也涉及到版本更新或者回退，要使用对应版本的代码）
4. 版本控制系统的分类
   - 分布式的版本控制系统：代码的版本分别在每个开发人员电脑上管理，管理好了之后代码共享放到git的服务器里面，实现代码共享（相互之间可以提交和获取代码）
   - 集中式版本控制系统：把代码的版本集中到一台服务器上管理svn

#### git的工作原理

1. 传统代码管理的原理：通过文件+日志文件来管理代码的版本
2. 使用git之类的工具管理：代码统一放到一个文件夹里面（会在当前要管理的代码文件夹里面创建一个.git文件夹（git本地仓库）） 代码的版本管理是在本地git仓库里面进行管理

#### git的基本使用

1. 了解git的一些常用命令

   - git add
   - git commit
   - git log
   - git status
   - git reset
   - git init

2. 安装git环境

   - 安装git:

     ​	brew install git

3. 创建ssh key、配置git

   - 设置username和email（github每次commit都会记录他们）

     ```python
     git config --global user.name 'greatman'
     git config --global user.email '18713048060@613.com'
     
     #name最好和GitHub上边的一样，email是一定要是注册GitHub的那个邮箱地址
     ```

   - 通过终端命令创建ssh key

     ```python
     ssh-keygen -t rsa -C '18713048060@163.com'
     
     #代码含义：
     -t 指定密钥类型，默认是 rsa ，可以省略。
     -C 设置注释文字，比如邮箱。
     -f 指定密钥文件存储文件名。
     ```

   - 接下来可以一路enter，使用默认文件名，那么就会生成id_rsa和id_rsa.pub两个秘钥文件

   - 接着会提示你输入两次密码

     ```python
     Enter passphrase (empty for no passphrase): 
     Enter same passphrase again:
     ```

     （该密码是你push文件的时候要输入的密码，而不是github管理者的密码）

     当然，你也可以不输入密码，那么push的时候就不需要输入密码，直接提交到GitHub上了

   - 接下来，就会显示如下代码提示

     ```python
     Your identification has been saved in /Users/macos/.ssh/id_rsa.
     Your public key has been saved in /Users/macos/.ssh/id_rsa.pub.
     The key fingerprint is:
     SHA256:0tgt189VrEvS9yRaMVrSaCBhTtrg2qUDqdPTbsqJKnA macos@Macs-MacBook-Pro.local
     ```

   - 使用命令：cat /Users/macos/.ssh/id_rsa.pub  ,复制里面的所有内容

     ```python
     ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDgqg70j5qebRotOHmMRs4CLe2qdqiB3hNwYrYrNrd5e7JguM3cwjIHBE8EVSfXCI0ud3DNc/lSefDH4EEXQ0bzHPIz6Z4DtY2PU1LtHwQe4X9qmUZw6xUN98sAZpWDD4z+TJYKs2PIhcfDhr9Bj3h5JLMWkUbGCTUA5OrYOHkO5TUN2zt8tsac8TdEX7lV4VRr7opSfrF3PUwg6NYPV+RLQoSrSH8ub6GGO/SPS7UaQwUMCK0EYmvYs/MshcDyhQxfuBF4/lk16Cn+/X77yKtWl42Sydf7Qtf87RPlA05qOpwCDyhwYF3vR49tNb6a+mk4Brc//JFqgBwxEzV4jdn/ macos@Macs-MacBook-Pro.local
     ```

   - 接下来，登录你的GitHub账号，主页中，右上角头像下拉，有一个settings，点进去，左边Profile列表中有一个SSH and GPG keys，点进入，有一个New SSH key,复制代码，创建一个SSH keys。

   - 验证你的GitHub连接

     ```python
     ssh -T git@github.com
     ```

   - git config --list

     如果正确的话会有下面的输出：

     ```python
     user.name=xxxxxx 
     user.email=xxxxxx@163.com
     ```

   - 在GitHub或GitLab上创建远程仓库

     登陆你的github，选择New Repository输入你要创建project的名字，这样代码库就创建完毕了，要记住你刚刚新建Repository的网址 

   - 本地git创建

     - 首先创建一个文件夹Git

       ```python
       mkdir ~/Git
       cd Git
       ```

     - 初始化该文件夹

     - ```python
       git init
       ```

       将会提示一下信息：

       nitialized empty Git repository in /Users/xxxxxxxxxxx 

     - 克隆项目

       ```python
       git clone '仓库的SSH url'
       #成功的话就能ls查看到仓库文件，cd进去会目录上会显示(master)主分支
       ```

     - 创建分支，一般不推荐直接在主分支上开发，所以创建自分支作为开发分支

       ```python
       git branch 分支名(创建分支)
       git checkout 分支名(切换分支)
       git checkout b 分支名(创建并切换到该分支)
       ```

     - 在进行了修改后，提交修改到缓存区

       ```python
       git add 目录/文件/.(.代表目录下的全部文件)
       ```

     - 如果本阶段修改已经完成，可以将修改提交到本分支（即可多次add后commit）

       ```python
       git commit -m '提交说明'
       ```

     - 查看修改的内容
	```python
	git diff 文件名
	```

     - 提交内容到远程仓库某个分支:

       ```python
       git push origin 分支名
       ```

     - 远程仓库中创建一个分支:

       ```python
       git checkout b 分支名
       git add .
       git commit -m '创建一个分支'
       git push origin 分支名
       ```

     - 删除远程分支：

       ```python
       git push origin :分支名     #分支名一定要和冒号紧挨着
       ```

     - **gitHub删除默认的master分支**

       ```python
       #如果用：
       
       git branch -D master;//删除本地master分支
       
       git push origin :master;//删除远程master分支
       
       会发现删除不了，因为在本地您处在master分支，在远程master为默认分支。
       
       解决之道：
       
       1.先建立自己的分支。
       
       git命令行：
       
       git branch temp;
       
       git push origin temp:temp;//将temp分支提交到远程分支上。
       
       2.在github上将master分支设置成不是default的分支，这里就要选择temp分支了，因为只有两个分支。
       
       github操作，点击后面的settings，选择不是master的分支为默认。
       
       3.再使用删除:（务必将当前分支切换到其他分支再删除）
       
       git branch -D master;//删除本地master分支
       
       git push origin :master;//删除远程master分支
       
       这样就完成删除了。
       
       4.如果你还想在将master分支做为默认的分支，再建一个叫master的分支，然后类似操作（将内容提交到master分支上，push到远程的github上，进入settings中设置master为默认的分支即可。）
       
       ```

     - 删除有两种方式：

       ```python
       1、工作区内"rm 文件名"，然后"git rm 文件名"，然后"git commit -m '备注'"
       2、直接使用"git rm 文件名"，然后"git commit -m '备注'"
       实验结果：
       1、工作区内"rm 文件名" 只是删除工作区内容，暂存区内容还是在的，在"git rm 文件名"操作之前可通过"git checkout --文件名"从暂存区进行恢复。
       2、"git rm 文件名"是既删除工作区内容也删除暂存区内容的，所以在"git commit -m '备注'"操作之前可以通过"git checkout HEAD -- 文件名"从版本库进行恢复，当然你要是直接用git reset HEAD^也行但是可能会影响你其他修改但是未提交的其他内容的。
       ```

     - 因为使用Git的目的就是多人协作开发，所以别人完成了的工作可能是你完成工作的基础，这时候你的本地就需要别人push到远程仓库（你们使用的是一个远程仓库）的代码 

       ```python
       git pull origin 分支名(拉取主仓库到本地本分支)
       ```

       上述过程其实相当于两步，

       git fetch origin （将远程仓库拉取到本地） 与  git merge origin 分支名 （合并本地与分支）

       **这样做其实是不太规范的**，规范的做法是：master主分支上存放远程仓库的所有版本，子分支只用来本地开发和上传修改，所以所有的pull都应当拉到master上，而不是直接到pull子分支，然后再通过merge master实现与master子分支合并：

       ```python
       $ git checkout master （从当前分支切换到主分支）
       
       $ git pull（拉取远程仓库当前版本到主分支）
       
       $ git checkout 子分支名 （从主分支切换到子分支）
       
       $ git merge master（与主分支合并）
       ```

       **注意：还有一个git rebase指令同样是合并分支，只是被合并的分支就消亡了，是完全的合并，而merge是内容合并。rebase可以与pull一起使用，这在第九点冲突时会说到。**

     - **在拉取之前，因为我们有本地的修改，它们还未保存，pull后会丢失，我们可以commit提交、版本合并后再拉取，但你可能还不想这样做，而且你的代码不一定到了上传、合并的时机，那就需要暂存我们的修改**

       ```python
       git stash(暂存修改)
       git stash pop(pull之后，弹出修改)
       ```

     - **在这个过程中，有很大可能遇到冲突**

       **最常见的冲突是内容冲突**：你修改了一个函数的实现，而远程仓库版本跟你的不一样，当主版本和你的修改合并时，这段代码的修改到底听谁的，所以就冲突了。通常出现在git pull与git merge的过程中。

       **这种冲突的解决方式是**：直接修改冲突文件，在执行指令后有冲突的文件会标识在命令行中，在冲突文件中有如下标志：

       ```python
       <<<<<<< HEAD
       
       这之间的是你的修改
       
       =======
       
       这之间是其他人的修改
       
       >>>>>>> 6853e5ff961e684d3a6c02d4d06183b5ff330dcc
       ```

       然后删除其他人的修改或者删除自己的修改，删除标识符，当处理完所有冲突后，执行git add与git commit即可。

     - **在git rebase的过程中，也许会出现冲突**： 在这种情况，Git会停止rebase并会让你去解决冲突；在解决完冲突后，用git add命令去更新这些内容的索引(index), 然后，你无需执行commit,只要执行:

       ```python
       $ git rebase --continue		#这样git会继续应用(apply)余下的补丁
       ```

       这里说的补丁是这种情况：

       表示把你的本地当前分支里的每个提交(commit)取消掉，并且把它们临时 保存为补丁(patch)(这些补丁放到".git/rebase"目录中),然后把本地当前分支更新为最新的"origin"分支，最后把保存的这些补丁应用到本地当前分支上

       ```python
       $ git rebase --abort	#在任何时候，你可以用--abort参数来终止rebase的行动，并且当前分支会回到rebase开始前的状态。
       ```

       #### Fork :从别人的GitHub复制项目到你的GitHub

       #### pull request:把你修改的项目推给别人

       #### clone：从GitHub上把项目克隆到本地 

       

     ## 廖雪峰的GitHub教程地址：https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374027586935cf69c53637d8458c9aec27dd546a6cd6000

最新创建的git本地push账号密码：同QQ
