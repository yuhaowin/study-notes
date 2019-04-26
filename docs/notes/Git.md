?> 持续更新ing。。。

git confing 文件有3个作用域

--local 对某一个仓库有效 （常用）

--system 对系统所有登陆用户有效（不用）

--global 对系统当前登陆用户有效（常用）

优先级 local > global > system

git config --global user.name ' '

git config --global user.email ' '

查看配置

git config --list --global

****

新建Git仓库

+ 将已经存在项目加入git管理：在项目文件夹内执行 git init
+ 新建项目并加入git管理：git init 项目名  会创建一个新的项目

****
git的工作区和暂存区

git  工作流程：修改多个文件 添加到暂存区 再一次提交

在工作区修改的内容，先添加到暂存区，确定修改内容后才提交到版本历史中。

暂存区具有承上启下的作用：
如果修改的内容合适，可以进一步变成一个正式的commit，如果内容不好，可以回退到未修改的工作区。

git add  可以添加多个文件/文件夹  用空格分割

git status 查看状态

git add -u  是将已经被git管理 但是发生了修改的 一起提交到暂存区
u = update

*******

git 重命名文件

直接在工作区修改了文件名：在git中的表现是删除了老的文件又新增的新的文件，可以通过git status 可以看到
在暂存区直接删除文件  git rm 文件名

+ 重命名步骤  工作区中文件重命名 ->  add新文件  -> rm旧文件

+ git mv 旧文件 新文件  可以直接代替上面3个步骤

*******

如果不想要暂存区中的修改了，将暂存区中修改的记录全部还原

git reset --hard  回退到某个commit 工作区和暂存区都和这个commit一致

git reset HEAD 的区别 取消暂存区的所有内容 保持暂存区和当前最新提交是一致的。


******

查看版本树变化过程

git log  只查看当前分支的所有commit

git log 分支名 查看某个分支的commit

git log --all 查看所有分支的commit

git log --all --graph 查看所有分支的commit 图形化界面

git log --oneline -n4  只看提交的注释  一共看4条

git --help --web log  网页查看log命令的帮助文档

******
查看当前版本库分支数

git branch -v  查看有多少个分支  git branch -av  -a参数是查看远程分支

git 创建分支时强烈建议本地分支和远端分支名称一致。

git创建的本地分支和远端分支关联的两种方法：

+ 本地分支dev和远端分支dev建立关联关系。

 `git branch --set-upstream-to=origin/dev dev`
 
+ 推送当前分支dev到远端并建立与远程分支dev的跟踪

 `git push --set-upstream origin dev`
 
 建议使用一下命令创建本地和远端关联的分支：
 
 基于远端分支dev的最新提交创建一个本地分支dev，并将本地分支和远端分支建立关联关系。
 
 `git checkout -b dev origin/dev`

git checkout -b temp 36cc4b4976be439910  基于本地某个提交版本创建一个分支

git 删除本地分支
git branch -d <BranchName>。-D 表示强制删除

git 删除远程分支
git push origin –-delete <BranchName>

*********

.git 内部

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g10f5fv859j30wi0qe44p.jpg)

git中的对象分为 commit  tree  blob

文件 HEAD 存放的是当前工作分支的引用，切换分支，HEAD内容也会变化。

文件 config 存放和该仓库相关的配置文件，只对本仓库生效，是优先级最高的配置。

文件夹 refs 存放里 heads（所有的分支）tags（所有的标签）

文件夹 objects 存放该仓库所有的对象

todo

[git diff](https://www.jianshu.com/p/80542dc3164e)

?> commit  tree  blob 三者之间的关系

任何文件的内容相同（即使文件名不同）也是同一个blob


将以往多个commit 合并成一个commit：


******

gitf 分支的合并

1、主干合并分支

进入分支，更新分支代码

（branch）git pull

切换主干

（branch）git checkout master

在主干上合并分支branch

（master）git merge branch --squash

提交合并后的代码

（master）git commit -m ‘合并备注’

将代码推送到远程仓库

（master）git push

2、分支合并主干

进入主干，更新主干代码

（master）git pull

切换分支

（master）git checkout branch

在分支上合并主干

（branch）git merge master --squash

提交合并后的代码

（branch）git commit -m ‘合并备注’

将代码推送到远程仓库

（branch）git push

[git合并参考资料](https://www.jianshu.com/p/684a8ae9dcf1)


******

分离头指针

意思是当前的变更没有和任何一个分支关联，

分离头指针的常常用于做一个尝试性的工作，并没有为改修改创建一个分支，随时准备丢弃掉

创建一个分离头指针的情况：

git checkout 544546

如果在544546这个提交基础上做了很多的修改，发现这些修改是有用的，可以通过为该commitid 创建分支的方法保留变更记录

git branch <新分支名> 544546

*****

修改最新提交commit的message

git commit --amend


*******

修改以前的commit的message（在本地完成，防止影响远端代码库）

git rebase -i 修改的commit的前一个commit

然后现在reword这种策略


*****

本地将多个commit合并成一个commit

然后现在squash这种策略

******

本地当前分支和暂存区内容比较

git diff --cached

*****

工作区和暂存区的内容比较

git diff

*****

比较两个commit之间的不同

git diff temp master -- <具体的文件>

****

取消暂存区内容修改

git reset HEAD

*****

取消工作区内容修改

git checkout <文件名>


git stash 

git stash list

git stash pop



接下来 视频 34


gitlab:

https://www.jianshu.com/p/b04356e014fa

https://blog.csdn.net/Z_Flank/article/details/80807458

https://blog.csdn.net/wq3028/article/details/81736486








