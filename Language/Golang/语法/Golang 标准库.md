Golang 的标准库非常丰富，涵盖了许多常用功能。

## 📦 **核心与基础库**

1. **fmt** - 格式化I/O
2. **os** - 操作系统功能（文件、进程、环境变量等）
3. **io** - 基础I/O接口
4. **bufio** - 带缓冲的I/O
5. **bytes** - 字节切片操作
6. **strings** - 字符串操作
7. **strconv** - 字符串与基本类型转换
8. **unicode** - Unicode相关功能
9. **errors** - 错误处理
10. **reflect** - 反射
11. **unsafe** - 不安全操作（慎用）

## 🕒 **时间与日期**

12. **time** - 时间和日期操作
13. **sync** - 并发同步（Mutex, WaitGroup, Once等）
14. **atomic** - 原子操作

## 🧮 **数学与随机数**

15. **math** - 基础数学函数
16. **math/rand** - 伪随机数生成
17. **crypto/rand** - 加密安全的随机数


## 📁 **文件与路径**

18. **path** - 路径操作（兼容不同OS）
19. **path/filepath** - 文件路径操作
20. **io/ioutil** - I/O工具函数（已部分弃用，推荐使用os和io包）
21. **os/exec** - 执行外部命令


## 🌐 **网络编程**

22. **net** - 网络基础
23. **net/http** - HTTP客户端和服务器
24. **net/url** - URL解析
25. **net/http/httptest** - HTTP测试
26. **context** - 上下文管理（用于超时、取消等）


## 🔐 **加密与安全**

27. **crypto** - 加密算法
28. **crypto/md5**, **crypto/sha1** - 哈希算法
29. **crypto/aes**, **crypto/rsa** - 加密算法
30. **encoding/base64** - Base64编码
31. **hash** - 哈希接口


## 📊 **数据编码**

32. **encoding/json** - JSON编码解码
33. **encoding/xml** - XML编码解码
34. **encoding/csv** - CSV文件处理
35. **encoding/gob** - Go二进制序列化
36. **encoding/binary** - 二进制编码


## 🧪 **测试与调试**

37. **testing** - 单元测试
38. **testing/quick** - 基于属性的测试
39. **flag** - 命令行参数解析
40. **log** - 简单日志
41. **runtime** - 运行时接口
42. **runtime/debug** - 调试功能


## 📦 **压缩与归档**

43. **compress/gzip**, **compress/zlib** - 压缩
44. **archive/tar**, **archive/zip** - 归档文件


## 📝 **模板处理**

45. **text/template** - 文本模板
46. **html/template** - HTML模板（带自动转义）


## 🔧 **其他重要库**

47. **sort** - 排序
48. **container** - 容器数据结构（heap, list, ring）
49. **database/sql** - 数据库接口
50. **embed** - 嵌入文件（Go 1.16+）
51. **go/ast**, **go/parser**, **go/token** - Go 代码解析
52. **plugin** - 插件系统（Linux/macOS）
