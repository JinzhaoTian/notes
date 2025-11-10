[Redis](https://redis.io/) 即远程字典服务，是一个开源的使用 ANSI C 语言编写、支持网络、可**基于内存**亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API ；Redis 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。提供了 Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang 等客户端，使用很方便。

- Redis：是非关系型数据库不仅可以做缓存，还能干其它事情。
- Memcache：是仅用做缓存。

常常让我们对这二者进行比较，主要也是由于 Redis 最广泛的应用场景就是 Cache。

## OUTLINE

![](_imgs/Pasted%20image%2020240329171612.png)

1. [基础架构](基础架构.md)：单线程，基于内存，I/O 多路复用
	- [Hash Slot](Hash%20Slot.md)
2. [安装部署](安装部署.md) 
3. [数据结构](数据结构.md) 
4. [应用场景](应用场景.md) 
	- [性能表现](性能表现.md)
	- [双写一致](双写一致.md)
5. [Redis 命令](命令.md) 
6. [相关问题](相关问题.md) 
7. [开发者指南](开发者指南.md) 



# Counterparts


1. [microsoft/garnet](https://github.com/microsoft/garnet)：Garnet，它实现了Redis协议，可以直接将Redis替换为Garnet，客户端不需要任何修改。
