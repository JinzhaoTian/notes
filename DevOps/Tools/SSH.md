SSH（Secure Shell Protocol）由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。SSH 是较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。SSH最初是UNIX系统上的一个程序，后来又迅速扩展到其他操作平台。SSH在正确使用时可弥补网络中的漏洞。

ssh 是安全的加密协议，用于远程链接 Linux 服务，ssh 默认端口是 22 ，安全协议版本 sshv2，出来 2 之外还有 1（有漏洞）。ssh 服务端主要包括两个服务功能 ssh远程链接和sftp服务，Linux ssh 客户端包括ssh远程链接命令，以及远程拷贝scp命令等。

### 服务器端设置

#### 安装服务

首先Linux服务器端要安装服务程序，因为ubuntu默认没有安装ssh server的，只有ssh client。可以运行下面命令一键安装：

```bash
$ sudo apt-get install ssh openssh-server
```



#### ssh免密登陆设置

我觉得免密登陆就是自动验证私钥公钥，具体方法就是，在客户端生成密钥对，然后将公钥传到服务器端（github也是这样用的）。首先在客户端生成密钥对：

```bash
$ ssh-keygen -t rsa -C tianjinzhao@sjtu.edu.cn
```

参数：

* -b：采用长度1024bit的密钥对, b=bits,最长4096，不过没啥必要  
* -t： rsa 采用rsa加密方式, t=type  
* -f： 生成文件名, f=output_keyfiles  
* -C： 备注，C=comment。就是pub key最后一段的用户信息。



然后将公钥（id_rsa.pub）传到服务器上：

```bash
$ ssh-copy-id -p 12347 username@10.211.55.3
```

传送好就可以免密登陆了，username是在服务器端给你分配的账号。免密登陆就是：

```bash
$ ssh -p 12347 username@10.211.55.3
```



### 客户端设置

对于Mac和Linux来说，Terminal上默认就有ssh，只需要使用命令登陆就好了，比如：

```bash
ssh 10.211.55.3            # 默认利用当前宿主用户的用户名登录
ssh parallels@10.211.55.3  # 利用远程机的用户登录
ssh parallels@10.211.55.3 -o stricthostkeychecking=no     # 首次登陆免输yes登录
ssh parallels@10.211.55.3 "ls /home/parallels"            # 当前服务器A远程登录服务器B后执行某个命令
ssh parallels@10.211.55.3 -t "sh /home/parallels/ftl.sh"  # 当前服务器A远程登录服务器B后执行某个脚本
```

然后输入账号密码就可以，其中parallels在本例中是服务器端已有的账户名，10.211.55.3是服务器端的ip地址，这个ip地址也可以是域名。

常见的ssh命令参数：

```bash
usage: ssh [-1246AaCfgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-e escape_char] [-F configfile]
           [-i identity_file] [-L [bind_address:]port:host:hostport]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-R [bind_address:]port:host:hostport] [-S ctl_path]
           [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]
```

退出登陆

```bash
$ logout
```



#### 补充：查询ip地址的方法：

```bash
$ ip addr show
# 或者
$ ip a
```



### SSH传输文件

ssh一般使用scp命令传输文件。

如从服务器端下载文件：

```bash
$ scp username@servername:/path/filename /var/www/local_dir/             # 下载文件
$ scp -r username@servername:/var/www/remote_dir/ /var/www/local_dir/    # 下载目录
```

将`filename`文件下载到本地目录` /var/www/local_dir`下。

上传本地文件到服务器：

```bash
$ scp -P 12347 /path/filename username@servername:/path/          # 上传文件，指定端口12347
$ scp -P 12347 -r local_dir/ username@servername:remote_dir/       # 上传目录
```



