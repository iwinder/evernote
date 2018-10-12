---
title: Docker小结
tags: Docker
grammar_cjkRuby: true
---
docker run -i -t <IMAGE_ID> /bin/bash：

-i：标准输入给容器   
-t：分配一个虚拟终端 
/bin/bash：执行bash脚本
-d：以守护进程方式运行（后台）
-P：默认匹配docker容器的5000端口号到宿主机的49153 to 65535端口
-p <HOT_PORT>:<CONTAINER_PORT>：指定端口号
- -name： 指定容器的名称
- -rm：退出时删除容器

docker stop <CONTAINER_ID>：停止container
docker start <CONTAINER_ID>：重新启动container
docker ps - Lists containers.
-l：显示最后启动的容器
-a：同时显示停止的容器，默认只显示启动状态

docker attach <CONTAINER_ID> 连接到启动的容器
docker logs <CONTAINER_ID>  : 输出容器日志

[Docker container常用的命令](https://blog.csdn.net/chajinglong/article/details/51536096)


## 保存镜像修改

获取容器id
```
docker ps -a
```
得到l类似：
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
498117d3b620        centos              "/bin/bash"         16 minutes ago      Exited (0) 7 minutes ago                       nifty_hermann
cc5b631df8b0        centos              "/bin/bash"         5 weeks ago         Exited (127) 5 weeks ago
```
提交保存修改
```
docker commit 498117d3b620 wind/hadoop
```
查看状态
```
docker images
```
得到：
```
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
wind/hadoop         latest              783f7509a46d        About a minute ago   200MB
centos              update3             e2c2530f5c3c        5 weeks ago          1.76GB
```

