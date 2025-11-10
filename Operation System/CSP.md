CSP（Communicating Sequential Process，通信顺序进程）是一种并发编程模型，用于描述两个独立的并发实体通过共享的通讯管道（Channel）进行通信的并发模型。

具体到编程语言，如 Golang，用到了 CSP 的很小一部分，即理论中的 Process/Channel（Golang 的 goroutine/channel）：这两个并发原语之间没有从属关系， Process 可以订阅任意个 Channel，Channel 也并不关心是哪个 Process 在利用它进行通信；Process 围绕 Channel 进行读写，形成一套有序阻塞和可预测的并发模型。