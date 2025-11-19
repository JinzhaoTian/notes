C++ 标准库的 `std::string` 是用来表示**字符串**的容器，但其实际上是**字节串**，并且不保留字符编码信息，以 `\0` 表示结束。

> [!tip] `std::string` 在某些场景下性能不够好
> 例如在涉及到系统 IO 的时候，通常会有如下接口：`std::size_t read(char* dst, std::size_t max_size);`，那么在为该接口分配缓冲区时，如果使用 `std::string`：
> ```cpp
> std::string buffer;
> buffer.resize(SIZE);
> auto read_size = read(buffer.data(), buffer.size());
> buffer.resize(read_size);
> ```
> 在某些场景（例如基础网络框架）下这么写会导致潜在的性能问题，可能会占用大量的CPU时间，因为 buffer 在 resize 的时候，不但**分配了内存还对这块内存做了初始化**，这导致了额外的 memset。


