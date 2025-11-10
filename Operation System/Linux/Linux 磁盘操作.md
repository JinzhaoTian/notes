
> 注意，挂载移动硬盘需要使用具有 root 权限的账号才可以。

### 核心概念

#### 分区（Partition）

- **定义**：分区是物理磁盘（如HDD、SSD）的逻辑划分，通过分区表（如MBR或GPT）将一块磁盘划分为多个独立区域。

- **作用**：隔离不同用途的数据（如系统分区、数据分区），或安装多个操作系统。

- **工具**：使用 `fdisk` 、 `gdisk` 或 `parted` 创建分区。


#### 物理卷（Physical Volume, PV）

- **定义**：物理卷是**LVM的基础存储单元**，可以是整个磁盘、一个分区，甚至是一个RAID设备，需通过 `pvcreate` 初始化为 PV 。

- **作用**：为 LVM 提供底层存储空间，后续可组合成卷组（VG）。


#### 卷组（Volume Group, VG）

- **定义**：卷组由一个或多个物理卷（PV）组成，是 LVM 中的存储池，VG 可以动态扩展或缩减。

- **作用**：统一管理多个PV的空间，供逻辑卷（LV）分配使用。


#### 逻辑卷（Logical Volume, LV）

- **定义**：逻辑卷是从卷组（VG）中划分出的虚拟块设备，类似于分区，但支持动态调整大小。

- **作用**：为文件系统提供灵活的存储空间（可动态扩展、缩减、快照等）。


### 文件系统（Filesystem）

- **定义**：文件系统是操作系统用于管理文件和目录的机制，需格式化在分区或逻辑卷上。

- **作用**：组织数据存储结构（如目录树、权限、元数据）。

- **常见类型**：ext4、XFS、Btrfs、NTFS（Windows兼容）。




### 查看信息


1. **查看移动硬盘信息**

```bash
sudo fdisk -l
```

![](_imgs/Pasted%20image%2020250526174018.png)


2. **查看磁盘和分区结构**

```bash
lsblk
```

![](_imgs/Pasted%20image%2020250527160051.png)




3. **查看 Volume group**

```bash
sudo vgdisplay
```

![](_imgs/Pasted%20image%2020250526174557.png)


3. **查看 Logical volume**
```bash
sudo lvdisplay
```

![](_imgs/Pasted%20image%2020250526174819.png)


### 硬盘分区


1. **分区操作**
```bash
sudo fdisk /dev/sda   # 使用fdisk（适用于MBR分区表）
sudo gdisk /dev/sda   # 使用gdisk（适用于GPT分区表）
```
- 常用命令（在 `fdisk` / `gdisk` 交互界面中）：
    - `n`：新建分区。
    - `d`：删除分区。
    - `p`：查看当前分区表。
    - `w`：保存并退出。
    - `q`：不保存退出。

![](_imgs/Pasted%20image%2020250527164515.png)

- **新建分区流**：
	- 输入 `p` 查看当前分区表，确认 `/dev/sda3`（或对应 LVM 分区）的起始扇区（Start）。
	- 输入 `d` 删除 `/dev/sda3`（**不会丢失数据，但谨慎操作！**）。
	- 输入 `n` 新建分区：
	    - 选择 `primary` 或 `extended`（取决于原分区类型）。
	    - **起始扇区必须和之前相同**（否则数据丢失！）。
	    - 结束扇区可以调整到最大可用空间（或手动输入大小）。
	- 输入 `t` 设置分区类型为 `8e`（Linux LVM）。
	- 输入 `w` 保存并退出。


2. **重新加载分区表** ：（**重要**）新建分区后需要重新加载分区表
```bash
sudo partprobe /dev/sda
```
- 通知OS内核分区表发生变化，重新读取磁盘分区表。



4. **创建物理卷**
```bash
sudo pvcreate /dev/sdb      # 将整个磁盘初始化为 PV
sudo pvcreate /dev/sda3     # 将某个分区初始化为 PV
```
- **使用场景**
	- 当你新增了一块硬盘（如 `/dev/sdb`）或一个新分区（如 `/dev/sda3`），并希望将其加入 LVM 管理时使用。
	- **首次设置 LVM 时必须使用**。


5. **扩展物理卷**
```bash
sudo pvresize /dev/sda3     # 调整 PV 大小以匹配底层分区
```

- **使用场景**
	- 当你调整了分区大小（如用 `fdisk` 扩展 `/dev/sda3`），但 PV 仍然显示旧的大小时使用。
	- **不适用于新设备**，仅适用于已加入 LVM 的 PV。



6. **扩容逻辑卷**
指定比例，
```bash
sudo lvextend -l +100%FREE /dev/mapper/centos-root
```
指定大小，
```bash
sudo lvextend -L +10G /dev/mapper/centos-root
```


7. **调整文件系统**

如果是 xfs 文件系统：
```bash
sudo xfs_growfs /dev/mapper/centos-root
```

如果是 ext4 文件系统：
```bash
sudo resize2fs /dev/mapper/centos-root
```


之后就可以看到 Size 有变化了，
```bash
df -h
```





### 挂载


4. **挂载硬盘**

```bash
mkdir ~/disk

sudo mount /dev/sda3 ~/disk
```

此时的 `disk` 文件夹对于普通用户是没有写操作的权限的，需要使用 `chmod` 命令修改文件夹权限，```
```bash
sudo chmod 777 ~/disk/
```
