1995 年 5 月，Brendan Eich 在 Netscape 内部完成的一个可运行引擎原型（当时叫 Mocha），而不是今天我们所知的 JavaScript 1.0 的全部。据 Eich 本人回忆，这个原型是在 1995年5月6日至15日之间完成的，他本人说过过：“引擎原型花了5月的十天”。

> [!quote] 被强加的“像 Java”约束
> Eich 最初被 Netscape 招募时的兴趣和计划，是将 Scheme 这样的函数式语言嵌入浏览器。然而，Netscape 与 Sun 的联盟改变了方向。
> 
> Eich 后来回忆说，他当时处于“市场命令”之下：语言必须“看起来像 Java，但不能大到僭越”，需要成为一种“傻乎乎的小兄弟语言”。具体而言，他被禁止使用 Java 那种基于类的面向对象语法。这个约束**直接决定了 JavaScript 底层形态的走向**：**既然不能用 `class`，他就必须找到另一种实现对象和复用的方式**。
> 
> 在无法使用类语法的前提下，Eich 将多种来源的思想压缩进了一个最小可运行模型中：
> - **语法外观**：借鉴 C 语言的基本语法，包括花括号、分号和保留字，以满足“看起来熟悉”的要求。
> - **对象模型**：将 C 的 struct、Smalltalk 的模式，以及 LISP 中数据与代码的对称性结合起来。最关键的选择是采用 Self 语言的原型（prototype）继承，以此替代 Java/C++ 的类继承。
> - **一等函数**：尽管 Scheme 的整体方案被放弃，但其一等函数的思想被保留了下来，成为 JavaScript 函数作为值传递、被存储和组合的基础。
> - **事件模型**：灵感来自 HyperCard，用于将事件绑定到 HTML 文档上。
> 
> 1995 年剩余的时间和整个 1996 年，Eich 的工作重心是**浏览器集成**（即后来被称为 **DOM Level 0** 的部分），以及引擎的持续修补。1996年，他还抽出时间**重写了引擎**，将原型中的部分实现替换为带有**垃圾回收**和**标记值**的 SpiderMonkey 架构。

1996 年 3 月，公开的 JavaScript 1.0 随 Netscape Navigator 2.0 发布，而第一个被 Eich 称为完整的版本则在 1996 年 8 月才出现。

> [!quote] 从 Mocha 到 JavaScript：命名的商业逻辑
> 这个语言在 Netscape 内部的代号是 Mocha，在 1995 年 9 月，随着 Netscape Navigator 2.0 的 Beta 版发布，它被更名为 LiveScript。在 1995 年 12 月，Netscape 与 Sun 联合宣布将其命名为 JavaScript。这次命名的动机是明确的商业考量：当时 Java 是炙手可热的技术，将新语言命名为“Java”的某种“脚本”形式，意在借势营销，将其定位为 Java 在网页端的轻量搭档。

1996 年，微软在 IE 3.0 中推出了自己的实现 JScript。由于 “JavaScript” 是 Netscape 与 Sun 的商标，微软无法直接使用该名称，但 JScript 在功能上模仿了 JavaScript，这意味着 Web 开发者面临两个行为可能不一致的实现。

为了阻止微软以自己的方言主导脚本语言，Netscape 于 1996 年 11 月将 JavaScript 提交给 Ecma International 进行标准化。标准化工作面临一个命名难题：不能使用 “JavaScript”（商标归属 Sun），也不能偏向 “JScript”（微软的方言）。最终，各方妥协采用了 ECMAScript 这个中性名称，对应的规范编号为 ECMA-262。第一版 ECMAScript 标准于 1997 年 6 月被 Ecma 大会采纳。从那时起，“JavaScript”作为语言的事实名称继续被广泛使用，而“ECMAScript”则成为其标准化规范的名字。







JavaScript 主要被作为客户端脚本语言在用户的浏览器上运行，不需要服务器的支持。可以嵌入HTML中，放在 `<body>` 标签内的 `<script>` 代码在页面载入时就会执行，且是按代码顺序（包括html、css代码）加载执行的。除非这段代码被定义在一个函数function里面，这个时候函数function被调用的时候才执行。

前端界面的组成有三个重要组成部分：
- HTML：从语义的角度，描述页面结构
- CSS：从审美的角度，描述样式（美化页面）
- JavaScript：从交互的角度，描述行为（提升用户体验）
JavaScript 是世界上用的最多的脚本语言，ECMAScript 在 2015 年 6 月，发布了 ECMAScript 6 版本语法标准。

一般来说，完整的 JavaScript 包括以下几个部分：
- ECMAScript，描述了该语言的语法和基本对象；
- 文档对象模型（DOM），描述处理网页内容的方法和接口；
- 浏览器对象模型（BOM），描述与浏览器进行交互的方法和接口。

JavaScript 常用来完成以下任务：
- 嵌入动态文本于 HTML 页面；
- 对浏览器事件作出响应；
- 读写 HTML 元素；
- 在数据被提交到服务器之前验证数据；
- 检测访客的浏览器信息；
- 控制 Cookie，包括创建和修改等；


## Overview

1. [JavaScript 类型系统](JavaScript%20类型系统.md)
	- [JavaScript Promise](JavaScript%20Promise.md)
	- [JavaScript Generators](JavaScript%20Generators.md)
2. [JavaScript 核心概念](JavaScript%20核心概念.md)
	- [JavaScript 原型链](JavaScript%20原型链.md)