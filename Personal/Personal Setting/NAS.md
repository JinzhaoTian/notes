NAS（Network Attached Storage，网络附加存储）是一种将存储设备直接连接到网络上的技术，使得网络中的多个用户和设备可以通过网络共享数据和文件。


## 群晖

[黑群晖NAS保姆级教程：手把手教您安装DS918+固件](https://post.smzdm.com/p/ar0v2on7/)

[DSM7.2 黑群晖安装教程 保姆喂饭级 附 2024 最新 ARPL 引导镜像 (mspace.cc)](https://www.mspace.cc/archives/1002)
## In Docker

有许多 NAS 系统可以运行在 Docker 容器中，利用 Docker 的灵活性和便携性来部署和管理 NAS 功能。
- OpenMediaVault（OMV）
- Nextcloud
- FreeNAS / TrueNAS Core


### TrueNAS Scale

![](_imgs/Pasted%20image%2020240604112326.png)

TrueNAS Open Storage 软件的三个版本，由 TrueNAS CORE、TrueNAS Enterprise 和 TrueNAS SCALE 组成。



### TrueNAS Core

在Docker中通过KVM来部署TrueNAS Core：

1. 构建环境
```dockerfile
# 使用一个基础镜像，例如Ubuntu
FROM ubuntu:20.04

# 安装必要的软件包
RUN apt-get update && apt-get install -y \
    qemu-kvm \
    libvirt-bin \
    virtinst \
    bridge-utils \
    cpu-checker \
    && rm -rf /var/lib/apt/lists/*

# 安装Docker工具包以便使用Docker CLI
RUN apt-get update && apt-get install -y docker.io

# 启动libvirtd
CMD ["/usr/sbin/libvirtd", "-d"]

```

2. 构建镜像
```bash
docker build -t truenas-kvm .
```

3. 启动容器
```bash
docker run --privileged --name truenas-kvm-container -d truenas-kvm
```

4. 下载TrueNAS Core ISO
```bash
docker exec -it truenas-kvm-container bash
cd /tmp
wget https://download.freenas.org/11.3/STABLE/x64/TrueNAS-11.3-U5.iso
```

5. 创建和启动虚拟机：在容器内创建一个虚拟机并使用下载的ISO镜像进行安装
```bash
# 创建虚拟机存储卷
qemu-img create -f qcow2 /var/lib/libvirt/images/truenas.qcow2 20G

# 使用virt-install命令创建并启动虚拟机
virt-install \
    --name truenas \
    --ram 4096 \
    --vcpus 2 \
    --os-type linux \
    --os-variant ubuntu18.04 \
    --hvm \
    --cdrom=/tmp/TrueNAS-11.3-U5.iso \
    --disk path=/var/lib/libvirt/images/truenas.qcow2,size=20 \
    --network network=default,model=virtio \
    --graphics vnc,listen=0.0.0.0

```

6. 访问TrueNAS Core安装界面：配置虚拟机后，你可以通过 VNC 客户端连接到虚拟机并完成TrueNAS Core 的安装。找到容器的IP地址和VNC端口，然后使用VNC客户端进行连接。

7. 完成TrueNAS Core安装：按照TrueNAS Core的安装向导完成安装。安装完成后，你可以通过容器的网络接口访问TrueNAS Core的Web管理界面。



### Nextcloud



## Unraid

![](_imgs/Pasted%20image%2020240810113101.png)

Unraid 不是免费的，它的收费模式是按照系统内的存储设备数量（不包含引导U盘）分为 Basic, Plus, Pro 三种，一次性买断制，价格分别为 59, 89, 129 （美元）。






## 硬件

QNAS Mini：

畅网 P5 N100 + 一体 SATA 线
AMS 1166 M.2 转 6SATA
8015风扇 x 2
SATA背板+配件
