?> 更新ing。。。

配置信息 local的优先级大于global

git config --global user.name ' '

git config --global user.email ' '


查看配置

git config --list --global

新建项目 加入 git管理

在项目文件夹内执行 git init

或者 git init 项目名  会创建一个新的项目

git add  可以添加多个文件  用空格分割

git status 查看状态


git add -u  是将已经被git管理 但是发生了修改的 一起提交到暂存区


git  工作流程：修改多个文件 添加到暂存区 再一次提交


直接在工作区修改了文件名：在git中的表现是删除了老的文件又新增的新的文件，可以通过git status 可以看到

在暂存区直接删除文件  git rm 文件名

*******

git 重命名文件

+ 重命名步骤  工作区中文件重命名 ->  add新文件  -> rm旧文件

+ git mv 旧文件 新文件  可以直接代替上面3个步骤

*******

查看版本树变化过程

git log  只查看当前分支的所有commit

git log --all 查看所有分支的commit

git log --all --graph 查看所有分支的commit 图形化界面

git log --oneline -n4  只看提交的注释  一共看4条

git --help --web log  网页查看log命令的帮助文档

******
查看当前版本库分支数

git branch -v  查看有多少个分支  git branch -av  -a参数是查看远程分支

git checkout -b temp 36cc4b4976be439910  基于默认提交版本创建一个分支


!> git reset --hard  将暂存区中修改的记录全部还原

git reset HEAD 的区别

*********

.git 内部

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g10f5fv859j30wi0qe44p.jpg)

git中的对象分为 commit  tree  blob

?> commit  tree  blob 三者之间的关系

任何文件的内容相同（即使文件名不同）也是同一个blob





将以往多个commit 合并成一个commit：











