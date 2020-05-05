# Docker Notes

## 删除 docker 容器

`语法：`

    docker rm CONTAINER_ID

`批量删除方法：` 想要删除 docker 镜像前，需要先删除对应的 docker 容器,。

    docker ps -a|awk '{print $1}'|xargs docker rm

## 查看/删除 docker 镜像     

查看镜像语法：

    docker images 

删除镜像语法：

    docker image rm image_id
