**Fail2Ban** 是一款开源的入侵防御工具，用于保护服务器免受暴力破解、恶意扫描和其他自动化攻击。它通过监控系统日志（如 `/var/log/auth.log`、`/var/log/nginx/error.log` 等），检测可疑行为（例如多次失败的登录尝试），并自动触发防火墙规则（如 iptables、nftables 或 firewalld）来封锁攻击者的 IP 地址。

**核心功能**

1. **自动封禁恶意 IP**
    - 当某个 IP 在短时间内多次触发失败事件（如 SSH 登录失败、Web 服务器暴力破解等），Fail2Ban 会临时或永久禁止该 IP 访问相关服务。
2. **支持多种服务**
    - 不仅适用于 SSH，还支持 Apache、Nginx、FTP、邮件服务（Postfix/Dovecot）、数据库等常见服务。
3. **灵活配置**
    - 通过自定义“过滤器”（正则表达式匹配日志）和“动作”（封禁规则），可适应不同场景。
4. **临时封禁与自动解封**
    - 默认封禁时间为 10 分钟（可调整），避免误封正常用户。

**工作原理**

1. **监控日志文件** ：Fail2Ban 持续读取服务的日志文件，例如：
    - SSH 日志：`/var/log/auth.log`（Debian/Ubuntu）或 `/var/log/secure`（CentOS/RHEL）。
    - Web 日志：`/var/log/nginx/error.log`。
2. **匹配攻击模式** ：使用预定义的“过滤器”（正则表达式）识别恶意行为，例如：
	- 多次 SSH 登录失败：`Failed password for .* from <IP>`。
3. **触发封禁动作** ：当同一 IP 在设定时间（`findtime`）内达到最大失败次数（`maxretry`），Fail2Ban 会调用防火墙添加一条规则封锁该 IP。