
使用shadowsocks连接

![](imgs/Pasted%20image%2020240719230812.png)


加密：chacha20

## Shadowsocks 服务端配置

1. 安装 shadowsocks-python
```bash
pip install shadowsocks
```

2. 编写配置参数
```bash
vi  /etc/shadowsocks.json
```

```json
{
    "server":"0.0.0.0",
    "server_port":16595,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"123456",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```


3. 开启防火墙
```bash
firewall-cmd --permanent --zone=public --add-port=16595/tcp
firewall-cmd --reload
```


4. 开启服务
```bash
ssserver -c /etc/shadowsocks.json -d start # 后台运行   
```


期间可能会出现错误，自行查看就行：

目前测试的加密方式
- [x] aes-256-cfb：用了一天被封
- [ ] chacha20