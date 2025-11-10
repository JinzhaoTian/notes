
1. **添加新用户**
```bash
sudo useradd [NAME]
```
**常用选项**：
- `-m`（`--create-home`）：自动创建用户的家目录（`/home/用户名`）
- `-d`：指定家目录路径（如 `-d /path/to/home`）
- `-s`：指定用户的默认 Shell（如 `-s /bin/bash`）
- `-G`：将用户添加到附加组（如 `-G wheel,developers`）
- `-u`：手动指定用户 UID（如 `-u 1005`）

**示例**：
```bash
sudo useradd -m -s /bin/bash testuser
```


2. **设置用户密码**：输入命令，然后输入两次密码
```bash
sudo passwd [NAME]
```

3. **将用户添加到附加组（可选）**
```bash
sudo usermod -aG wheel [NAME]
```
**管理员权限**：
- **在 Ubuntu/Debian 中（使用 `sudo` 组）**
```bash
sudo usermod -aG sudo [NAME]
```

- **在 CentOS/RHEL 中（使用 `wheel` 组）**
```bash
sudo usermod -aG wheel [NAME]
```


4. **验证用户信息**
```bash
id [NAME]
```
输出类似：
```
uid=1001(testuser) gid=1001(testuser) groups=1001(testuser),10(wheel)
```

5. **切换到新用户**
```bash
su - [NAME]
```

6. **删除用户**
```bash
sudo userdel -r [NAME]  # -r 表示同时删除家目录和邮件池
```



## Linux 用户组

Linux 用户组主要分为两类：
1. **主组（Primary Group）**
	- 每个用户必须有且只有一个主组，默认与用户名相同（如用户 `testuser` 的主组是 `testuser`）。
	- 用户创建的文件默认属于其主组。

2. **附加组（Supplementary Groups）**
	- 用户可以被加入多个附加组，从而获得额外的权限（如 `sudo`、`docker`、`www-data` 等）。


### 常见系统用户组

|用户组|用途|
|---|---|
|`root`|超级管理员组，拥有系统最高权限。|
|`sudo`|允许用户通过 `sudo` 执行管理员命令（Ubuntu/Debian 默认使用此组）。|
|`wheel`|功能同 `sudo`，CentOS/RHEL 默认使用此组。|
|`adm`|允许查看系统日志（如 `/var/log/`）。|
|`docker`|允许用户直接操作 Docker（无需 `sudo`）。|
|`www-data`|Web 服务器（如 Nginx/Apache）运行时的默认用户组。|
|`users`|普通用户组（默认组）。|
