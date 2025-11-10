## Bash编程

> 参考：[阮一峰的网络日志](https://wangdoc.com/bash/)，[Bash编程入门](https://www.jianshu.com/p/e1c8e5bfa45e)

在类Unix上的shell是可以用来执行系统调用和系统命令的，一般默认运行一种能给用户提供命令行环境的程序，如bash、csh、zsh等等，其中Linux一般默认是bash，我的mac则默认是zsh。



## 终端

> 参考：[博客](https://www.jianshu.com/p/a891af6f87e0)

操作系统分为两个部分，一部分称作内核，另一部分成为用户交互界面，终端就是连接内核与交互界面的这座桥。反正shell是一个程序，分辨一下自己使用的是哪个程序，然后使用就行了。

### Bash

**Bash的默认配置文件是`~/.bash_profile`**，是Linux的默认shell，由 GNU 组织开发。

### Zsh

**Zsh的默认配置文件是`~/.zshrc`**，兼容 bash，还有自动补全等好用的功能，使用起来更加优雅。

### 相互切换

可以在命令行切换，切换命令如下：

```Bash
chsh -s /bin/bash   # 切换到bash
chsh -s /bin/zsh   # 切换到zsh
```



## 配置文件

>  参考：[博客](https://www.jianshu.com/p/35ad1b375e50)，[/etc/profile详解](https://blog.csdn.net/qq_32457341/article/details/104452084)，[Bash和Zsh配置文件执行顺序详解](https://shreevatsa.wordpress.com/2008/03/30/zshbash-startup-files-loading-order-bashrc-zshrc-etc/)

常常在碰到环境配置问题的时候，往往要修改一堆配置文件，比如 `/etc/profile`，`.bashrc`，或者 `.zshrc` 等等，就一直不懂这些是怎么配置运行的，现在来理一理。

### Bash

Bash有几种不同的运行模式，login shell与non-login shell，interactive shell与non-interactive shell（比如执行shell脚本）。这两种分类方法是交叉的，也就是说一个login shell可能是一个interactive shell，也可能是个non-interactive shell。

login shell与non-login shell的**主要区别**在于它们**启动时会读取不同的配置文件，从而导致环境不一样**。



**配置文件的执行顺序为**：

* **Interactive login**：`/etc/profile`→ `(~/.bash_profile | ~/.bash_login | ~/.profile)`→`~/.bash_logout`。其中，`~/.bash_profile`、 `~/.bash_login` 和 `~/.profile` 这三个文件只会读取一个，按照这个顺序读取，读一个就行。
* **Interactive non-login**：`/etc/bash.bashrc` →`~/.bashrc` 。注意在Linux里面是`/etc/bash.bashrc`文件，但是在 macOS 里面好像就是直接是`/etc/bashrc`。
* **Script**：`BASH_ENV`



Interactive non-login模式是最常用的。



**文件详解**：

* `/etc/profile`：这个文件里面存储了类Unix系统中配置的一些全局变量，这些变量对所有的用户可以用。当单个用户登陆系统时，首先会加载 `/etc/profile` 中的全局变量的信息。在有的时候，我们采用一些其他的方式登录时，有些全局变量是不起作用的，这个时候需要执行下面的语句：

  ```bash
  source /etc/profile
  ```

  就可以将全局变量刷新。

* `/etc/bashrc`or`/etc/bash.bashrc`：为每一个运行bash shell的用户执行此文件。当bash shell被打开时,该文件被读取（即每次新开一个终端，都会执行bashrc）。

* `~/.bash_profile`：每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次。默认情况下,设置一些环境变量,执行用户的`.bashrc`文件。

* `~/.bash_login`：

* `~/.profile`：

* `~/.bashrc`：该文件包含专用于你的bash shell的bash信息，当登录时以及每次打开新的shell时,该该文件被读取。

* `~/.bash_logout`：当每次退出bash shell时,执行该文件



### Zsh

**配置文件的执行顺序为**：

* **Interactive login**：`/etc/zshenv`→ `(~/.zshenv ` → `/etc/zprofile` → `~/.zprofile` → `/etc/zshrc` → `~/.zshrc` → `/etc/zlogin`  →  `~/.zlogin` → `~/.zlogout` → `/etc/zlogout`
* **Interactive non-login**：`/etc/zshenv`→ `(~/.zshenv ` → `/etc/zshrc` 
* **Script**：`/etc/zshenv`→ `(~/.zshenv `



## 编程语法

 


