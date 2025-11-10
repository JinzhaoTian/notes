CUDA（Compute Unified Device Architecture）是由NVIDIA开发的一种并行计算架构和编程模型。它使得开发者能够利用NVIDIA图形处理单元（GPU）的强大计算能力，进行高效的并行计算。

具体来说，CUDA允许开发者在GPU上执行计算密集型任务，如科学计算、图像处理、机器学习等。通过CUDA，开发者可以编写与C语言类似的代码，称为CUDA C/C++，来执行并行计算任务。

**关键特点**：

1. **并行计算能力**：GPU拥有大量的处理核心，可以同时处理大量的线程，适合处理并行计算任务。
2. **编程模型**：CUDA提供了一个简单易用的编程模型，允许开发者将计算任务分配到GPU上进行处理。
3. **高性能**：由于GPU的并行处理能力，使用CUDA可以显著提高计算性能，尤其在处理大规模数据集和复杂计算时表现尤为突出。
4. **兼容性**：CUDA支持多种编程语言和工具，如CUDA C/C++、Fortran、Python（通过NVIDIA的库）等。

## 架构

![](imgs/Pasted%20image%2020240203173052.png)

CUDA 编程模型是一个异构模型，需要 CPU 和 GPU 协同工作。在 CUDA 中，**host** 和 **device** 是两个重要的概念，我们用 host 指代 CPU 及其内存，而用 device 指代 GPU 及其内存。CUDA 程序中既包含 host 程序，又包含 device 程序，它们分别在 CPU 和 GPU 上运行。同时，host 与 device 之间可以进行通信，这样它们之间可以进行数据拷贝。

![](imgs/Pasted%20image%2020240203173708.png)

典型的 CUDA 程序的执行流程如下：
1. 分配 host 内存，并进行数据初始化；
2. 分配 device 内存，并从 host 将数据拷贝到 device 上；
3. 调用 CUDA 的核函数在 device 上完成指定的运算；
4. 将 device 上的运算结果拷贝到 host 上；
5. 释放 device 和 host 上分配的内存。

