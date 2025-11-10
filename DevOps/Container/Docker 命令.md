
* `systemctl start docker`：启用docker daemon
* `docker image ls`：查看镜像列表
* `docker container ls`：查看容器列表
* `docker ps -l`：查看容器的状态
* `docker search [name]`：查找镜像
* `docker image pull [name]`：拉取镜像
* `docker image rm [name]`：删除镜像
* `docker run -i -t [name] [command]`：以交互终端的方式启用 `[name]` 镜像，启用时运行命令`[command]` 
* `docker rm [name]`：删除容器
* `docker rmi [name]`：删除镜像
* `docker start -i -a [name]`：启动容器，交互式，依附于终端
* `docker stop [name]`：停止容器

  