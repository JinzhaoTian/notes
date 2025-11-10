End-to-End Argument 是计算机系统设计中的一项核心原则，最早由 Saltzer、Reed 和 Clark 在 1981 年的论文《[End-to-End Arguments in System Design](https://web.mit.edu/Saltzer/www/publications/endtoend/endtoend.pdf)》中提出。

![](_imgs/Pasted%20image%2020250715094749.png)

它强调系统的某些功能（如可靠性、安全性等）**应当由通信的终端（End）而非中间层（如底层网络）来实现**，因为中间层的保证往往无法完全满足终端的需求。

简单的抽象来说，从系统分层的角度，高层的“可靠”，“顺序”，“重复”的概念和低层比如 TCP 的“可靠”，“”顺序“，“去重”不是一回事儿。**系统设计应考虑高层系统语意，而以低层语意为工具，而系统的真实想要达成的意图比工具要重要的多。**




## 参考

1. [End to End Argument(可能是最重要的系统设计论文) - 知乎](https://zhuanlan.zhihu.com/p/55311553)


