
Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

![](imgs/Pasted%20image%2020240702175911.png)

> Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。


`dockerfile` 编写完之后，执行构建动作
```bash
docker build -t nginx:v3 .
```



# 封装环境镜像


[Docker 封装anaconda环境，生成镜像并打包，纯小白一文读懂（二）_anaconda 的 docker 镜像构建-CSDN博客](https://blog.csdn.net/qq_32101863/article/details/120344080)



# 离线构建镜像


内网环境没法 pull 镜像，但是 docker 本身可以将已有的镜像导出成 tar 文件，并且可以再次导入到docker，利用这一点，可以实现离线镜像文件的下载。

1. 使用命令将镜像文件导出：
```bash
docker save java:8 -o java.tar      # 将java 8的镜像导出成tar文件
```

2. 将 tar 文件上传到内网 docker 服务器。
3. 使用命令导入镜像文件：
```bash
docker load -i java.tar
```

4. 查看导入的镜像文件：
```bash
docker images
```
