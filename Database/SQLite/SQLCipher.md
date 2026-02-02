SQLCipher 是一个开源的数据库加密扩展，用于对 SQLite 数据库文件进行透明的 256 位 AES 加密。


## 主要特性

1. **透明加密**
	- 使用相同的 SQLite API，无需大幅修改代码
	- 加密/解密在读写时自动完成
	- 支持所有 SQLite 功能

2. **安全性**
	- 256 位 AES 加密（支持 CBC 和 GCM 模式）
	- 完整的数据库文件加密（包括元数据和日志）
	- 基于密码的密钥推导（PBKDF2）
	- 可选的 HMAC 完整性检查

3. **平台支持**
	- iOS 和 Android（移动端常用）
	- macOS、Linux、Windows
	- 多种编程语言绑定

## 使用方式

```c
// 传统 SQLite
sqlite3_open("test.db", &db);

// SQLCipher
sqlite3_open("test.db", &db);
sqlite3_key(db, "secret-key", 10);  // 设置加密密钥
```

