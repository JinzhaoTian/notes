服务器 CPU 架构包括 x86 、 ARM 和 MIPS 等， x86 为当前服务器 CPU 主流架构，几乎占据目前服务器全部市场份额，代表性厂商为 Intel 和 AMD 。

- x86
	- 国外：Intel、AMD
	- 国内：海光、兆芯、申威
- ARM
	- 国外：高通、Cavium、Amazon
	- 国内：华为海思、飞腾
- MIPS
	- 龙芯

![](_imgs/Pasted%20image%2020240628152403.png)

## 指令集

### RISC

精简指令集计算（Reduced Instruction Set Computing，RISC），结构简单，选取了使用频率高的简单指令，指令长度固定，每条指令执行的时间通常是一个时钟周期。

RISC-V 是基于 RISC 原理建立的免费开放指令集架构（Instruction Set Architecture，ISA），V 是罗马字母，代表第五代 RISC（精简指令集计算机），可读作 RISC-FIVE 。


#### 处理器历史

- **1981年**：IBM 801——IBM研究院开发的原型RISC处理器，首次验证了RISC理念的可行性。
- **1984年**：MIPS R2000——由MIPS计算机系统公司发布的第一款商用RISC处理器，广泛应用于嵌入式系统。
- **1986年**：SPARC（Scalable Processor Architecture）——由Sun Microsystems开发，用于工作站和服务器。
- **1987年**：ARM1——Acorn Computers开发的第一款ARM处理器，用于计算机和嵌入式系统。
- **1990年**：IBM POWER1——IBM发布的第一款POWER架构处理器，用于高性能计算。
- **1991年**：DEC Alpha——由Digital Equipment Corporation发布的64位RISC处理器，性能优越。
- **1993年**：PowerPC 601——由IBM、Apple、Motorola联合开发的PowerPC架构处理器，用于Apple的Macintosh计算机和嵌入式系统。
- **1996年**：ARM7——ARM公司发布的低功耗RISC处理器，广泛应用于移动设备。
- **2002年**：IBM POWER4——IBM发布的高性能RISC处理器，用于服务器和超级计算机。
- **2003年**：ARM Cortex-A8——ARM公司发布的新一代高性能RISC处理器，广泛应用于智能手机和平板电脑。
- **2006年**：SPARC T1（Niagara）——由Sun Microsystems发布的多线程RISC处理器，优化了服务器性能。
- **2011年**：ARM Cortex-A15——ARM公司发布的高性能多核处理器，应用于高端智能手机和平板电脑。
- **2013年**：IBM POWER8——IBM发布的高性能RISC处理器，用于数据中心和高性能计算。
- **2016年**：**RISC-V**——由加利福尼亚大学伯克利分校开发的开源RISC架构，获得广泛关注和应用。
- **2021年**：Apple M1——苹果公司发布的基于ARM架构的高性能处理器，广泛应用于Mac和iPad产品线。
- **2022年**：IBM POWER10——IBM发布的最新一代高性能RISC处理器，进一步提升了服务器和数据中心的性能。


华为鲲鹏、飞腾都获得了 Arm V8 永久授权。

#### 处理器产品

1. **ARM 处理器**
    - **ARM Cortex-M 系列**：用于低功耗嵌入式系统，如微控制器和传感器。
    - **ARM Cortex-A 系列**：用于高性能移动设备，如智能手机和平板电脑（如苹果的 iPhone 和 iPad）。
    - **ARM Cortex-R 系列**：用于实时处理系统，如汽车电子和工业控制。

2. **MIPS 处理器**
    - **MIPS R2000**：早期的 MIPS 处理器，用于嵌入式系统和工作站。
    - **MIPS32/64**：用于现代网络设备和嵌入式系统。

3. **SPARC 处理器**
    - **SPARC T 系列**：用于多线程处理的服务器和数据中心。
    - **UltraSPARC**：用于高性能工作站和服务器。

4. **PowerPC 处理器**
    - **PowerPC G 系列**：早期用于苹果的 Macintosh 计算机。
    - **PowerPC 4xx 系列**：用于嵌入式应用，如汽车电子和网络设备。
    - **PowerPC 750（G3）**：用于任天堂的游戏机，如 GameCube 和 Wii。

5. **IBM POWER 处理器**
    - **IBM POWER8**：用于高性能计算和数据分析。
    - **IBM POWER9**：用于人工智能和机器学习应用。

6. **RISC-V 处理器**
    - **SiFive Freedom 系列**：开源 RISC-V 处理器，用于嵌入式和物联网应用。
    - **SweRV**：由 Western Digital 开发的开源 RISC-V 处理器，用于存储控制器和嵌入式应用。


### CISC

复杂指令集计算（Complex Instruction Set Computing，CISC）试图最小化每个程序的指令数量，从而牺牲每个指令的周期数量。

基于CISC体系结构的计算机旨在降低内存成本。因为，大型程序需要更多的存储空间，因此增加了内存成本，并且大型内存变得更加昂贵。为了解决这些问题，可以通过将操作数量嵌入单个指令中来减少每个程序的指令数量，从而使指令更加复杂。侧重于硬件执行指令的功能性，CISC指令及处理器的硬件结构复杂，CISC指令复杂，指令长度与周期不固定，在处理能力上有优势。



#### 处理器历史

- IBM System/360（1964年）：采用了CISC架构，并且引入了一个统一的指令集，适用于不同的应用场景。
System/360的设计思想是通过复杂的指令集和丰富的寻址模式，来支持多种数据类型和操作，从而简化软件开发。

- **Intel 8086**（1978年）：Intel 8086是CISC架构在个人计算机领域的代表。它采用了复杂的指令集，可以直接进行字符串操作、内存寻址等复杂操作。8086及其后续产品（如80286、80386）在PC市场上取得了巨大的成功，确立了CISC在个人计算机领域的地位。


- 现代CISC处理器（2000s至今）：现代CISC处理器（如Intel的x86架构处理器）结合了RISC的设计理念，通过超标量、超流水线、多级缓存等技术，提高了执行效率。虽然底层仍然是CISC架构，但很多现代处理器在内部使用RISC-like的微架构来实现复杂指令的高效执行。

## ARM 发展历史

ARM（原名为 Acorn RISC Machine，现为 Advanced RISC Machines）的发展历史可以追溯到20世纪80年代。

- **1985年**：Acorn Computers 发布了首款 ARM 处理器，即 ARM1。这是第一个成功实现的 RISC 处理器之一。
- **1987年**：ARM2 处理器发布，比 ARM1 提供了更高的性能和更低的功耗，被用于 Acorn Archimedes 计算机。
- **1990年**：Acorn Computers、Apple 和 VLSI Technology 联合成立了 ARM Holdings，专门开发和授权 ARM 架构。
- **1991年**：ARM6 处理器发布，这是 ARM Holdings 成立后推出的第一个处理器。
- **1993年**：ARM7 处理器发布，成为当时最广泛使用的 ARM 处理器之一。
- **2002年**：ARM9 和 ARM10 处理器发布，进一步提升了性能，并在消费电子和嵌入式系统中广泛应用。
- **2004年**：ARM Cortex 系列处理器发布，标志着 ARM 处理器进入了一个新的时代。Cortex-M 系列专注于微控制器市场，Cortex-R 系列用于实时处理，Cortex-A 系列则面向高性能应用。
- **2005年**：ARM11 处理器发布，成为首款用于智能手机的 ARM 处理器之一。
- **2011年**：ARMv8-A 架构发布，支持 64 位计算。这是 ARM 历史上的一个重大里程碑，为移动设备、服务器和高性能计算开辟了新的市场。
- **2012年**：首款基于 ARMv8 架构的处理器 Cortex-A53 和 Cortex-A57 发布。
- **2013年**：苹果发布的 A7 处理器（用于 iPhone 5s）成为首款基于 ARMv8 64 位架构的移动处理器。
- **2016年**：ARM Cortex-A73 和 Cortex-A35 处理器发布，进一步优化了性能和能效。
