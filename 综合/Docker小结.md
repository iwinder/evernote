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