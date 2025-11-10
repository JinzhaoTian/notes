Mustache 是一个模板引擎库，header-only、无依赖，用于 C++ 11 及以上版本，项目地址：[kainjow/Mustache](https://github.com/kainjow/Mustache)。 

在模板文件中，Mustache 通过预定义的标签（tags）来完成所有数据渲染工作，这样的设计强制地将业务逻辑和表现层分离开，让模板更加清晰和易于维护。

## 基本语法

1. **变量替换**：`{{variable}}` 会将指定变量值进行 HTML 转义后输出。
2. **保留HTML**：`{{{variable}}}` 或 `{{&variable}}` 会输出变量的原始内容，不进行 HTML 转义。
3. **区块（Sections）**：`{{#section}} ... {{/section}}` 根据变量的类型进行不同操作：
	- 如果变量是数组或列表，它会循环遍历其中的每一个元素；
	- 如果变量是布尔值，则可以根据其真或假来决定是否显示该区块的内容。
4. **反区块（Inverted Sections）**：`{{^section}} ... {{/section}}` 仅在变量值为 `null`、`undefined`、`false` 或空列表时，才会渲染输出该区块的内容。
5. **注释**：`{{! This is a comment }}` 注释内容不会被渲染到最终输出中。
6. **部分模板（Partials）**：`{{>partial}}` 用于引入其他外部模板文件，实现模板的复用。


