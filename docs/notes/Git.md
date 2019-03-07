?> 待更新。。。

配置信息 local的优先级大于global

git config --global user.name ' '

git config --global user.email ' '


查看配置

git config --list --global

新建项目 加入 git管理

在项目文件夹内执行 git init

或者 git init 项目名  会创建一个新的项目

命令 git log  查看提交记录 


git add  可以添加多个文件  用空格分割

git status 查看状态


git add -u  是将已经被git管理 但是发生了修改的 一起提交到暂存区


git  工作流程：修改多个文件 添加到暂存区 再一次提交


直接在工作区修改了文件名：在git中的表现是删除了老的文件又新增的新的文件，可以通过git status 可以看到

在暂存区直接删除文件  git rm 文件名

重命名步骤  工作区中文件重命名 ->  add新文件  -> rm旧文件

git mv 旧文件 新文件  可以直接代替上面3个步骤



git log --oneline -n4  表示只看提交的注释  一共看4条


git branch -v  查看有多少个分支  git branch -av ？？

git checkout -b temp 36cc4b4976be439910  基于默认提交版本创建一个分支


*********


将以往多个commit 合并成一个commit：











