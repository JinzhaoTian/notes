![](_imgs/Pasted%20image%2020240401151333.png)

[PM2](https://pm2.keymetrics.io/docs/usage/quick-start/) 是一个守护进程管理工具，帮助您管理和守护您的应用程序。

```
npm install -g pm2
```

# 命令

1. 启动应用：`pm2 start app.js`
2. 列出应用：`pm2 [list|ls|status]`
3. 查看日志：`pm2 logs --lines 200`
4. 查看监控信息：`pm2 monit`
5. 停止进程：`pm2 stop [all/0]`
6. 重启进程：`pm2 restart [all/0]`
7. 删除进程：`pm2 delete [all/0]`
