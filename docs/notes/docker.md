docker 

docker images   显示本地有的镜像

docker pull + 镜像名称   从docker hub上面拉取镜像

docker run
   --name 定义容器的名称

   -d 让docker容器在后台运行到

   -a 查看已经创建的容器

   -s 查看启动的容器

docker start [NAME]/[CONTAINER ID]  启动名称为docker_name的容器

docker stop [NAME]/[CONTAINER ID]   关闭名称为docker_name的容器

docker rm  [NAME]/[CONTAINER ID]    删除名称为docker_name的容器

docker rmi [NAME]/[IMAGE ID]        删除名称为docker_name的镜像
 
docker rename old_name new_name     给容器重命名

如果删除时遇到依赖问题导致无法删除，可以查询依赖关系

docker image inspect --format='{{.RepoTags}} {{.Id}} {{.Parent}}' $(docker image ls -q --filter since=xxxxx)

xxxxx 为无法删除的 image-id

docker ps: 查看当前运行的容器
docker ps -a:查看所有容器，包括停止的。


************

docker 下载 mysql images

docker pull mysql

docker run --name docker-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345678 -d mysql

进入 docker-mysql容器: docker exec -it docker-mysql bash

登陆 mysql

mysql -uroot -p12345678

为root分配权限，以便可以远程连接：

mysql> grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;

mysql> flush privileges;

因为Mysql5.6以上的版本修改了Password算法，这里需要更新密码算法，操作步骤如下：

mysql> ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

退出 容器 exit

***************

查找 docker 容器在主机内部的 ip 与端口

docker network inspect bridge
