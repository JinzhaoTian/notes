## 配置文件加载顺序

Linux 中，Shell 分为两种类型：

| Shell 类型                    | 触发方式                | 读取的配置文件                                      |
| --------------------------- | ------------------- | -------------------------------------------- |
| Login Shell                 | SSH 登录、bash --login | /etc/profile → ~/.profile（或 ~/.bash_profile） |
| Non-login Interactive Shell | 打开新终端、bash          | ~/.bashrc                                    |

**关键区别**：SSH 登录时启动的是 **Login Shell**，它读取 `~/.profile`，**不读取 `~/.bashrc`**。

而 Debian/Ubuntu 的 `~/.profile` 默认只加载 `~/bin` 和 `~/.local/bin`，不会加载 `~/.npm-global/bin`。
