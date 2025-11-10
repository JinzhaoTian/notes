
## 在线安装

通过[官网](https://www.docker.com/)来安装


## 离线安装

1. 查看操作系统版本
```bash
cat /proc/version
or
uname -a
```

2. 查看操作系统架构
```bash
arch
or
uname -m
```

3. [下载 Docker 离线包](https://download.docker.com/linux/static/stable/) 
4. [下载Docker Compose 离线包](https://github.com/docker/compose/releases/) 

5. 解压安装包
```bash
tar -xvf docker-18.06.3-ce-aarch64-linux.tar.gz -C /usr/local/  
tar -xvf docker-compose-aarch64_linux-1.29.2.tar -C /usr/local/
```

6. 配置环境变量
```bash
export PATH=/usr/local/bin:$PATH

source ~/.bashrc
```

7. 启动Docker服务
```bash
systemctl start docker     # 启动Docker服务
systemctl enable docker    # 设置为开机自启
```

8. 验证安装
```bash
docker version
docker-compose —version
```

[docker离线安装并导入镜像「建议收藏」-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2157725)

