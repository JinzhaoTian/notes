---
tags:
  - obsidian
---

## 语法


### callout

callout 是 obsidian 的特有语法，源于 [Markdown](../../Language/Data%20Format/Markdown.md) 的引用：
```markdown
> [!note] Title
> Content
```
分为四个部分：
- 开头的 `>`
- 标明 callout 类型的 `note`
- 标题 `Title`
- 正文 `Content`


有若干种类型：
- `note`：
> [!note] Title
> Content

- `abstract`, `summary`, `tldr`
> [!summary] Title
> Content

- `info`
> [!info] Title
> Content

- `todo`
> [!todo] Title
> Content

- `tip`, `hint`, `important`
> [!tip] Title
> Content

- `success`, `check`, `done`
> [!success] Title
> Content

- `question`, `help`, `faq`
> [!faq] Title
> Content

- `warning`, `caution`, `attention`
> [!warning] Title
> Content

- `failure`, `fail`, `missing`
> [!failure] Title
> Content

- `danger`, `error`
> [!error] Title
> Content

- `bug`
> [!bug] Title
> Content

- `example`
> [!example] Title
> Content

- `quote`, `cite`
> [!quote] Title
> Content


## 核心插件


### Properties

Properties 可用于整理笔记的相关信息，其包含结构化数据，例如文本、链接、日期、复选框和数字。属性还可以与社区插件结合使用，这些插件可以对您的结构化数据进行一些实用的操作。

#### 创建

有以下几种方法可以为笔记添加 Properties：
1. 使用 `Add file property` 命令
2. 使用 `Cmd/Ctrl+` 快捷命令
3. 在文件的最开始位置输入 `---` 

添加后，在文件开头会出现 Properties，其每个属性行需指定一个键值对：Property 的名字和值。

#### 内置 Properties

- `tags`
- `cssclasses`
- `aliases`

#### 美化

```css
/* hides "Properties" title, also hides dropdown */
.metadata-properties-heading {
    display: none;
}

.metadata-properties,
.metadata-properties *,
.metadata-properties-heading {
    font-family: "霞鹜文楷", "PingFang SC", "Microsoft YaHei", var(--font-monospace), "Consolas", "Source Code Pro";
}

body {
    --metadata-label-font-weight: 500;
    --metadata-label-text-color: #bdeec;
    --metadata-label-width: max(22.5%, 8rem);
    --metadata-gap: 0;
    --metadata-padding: 2;
    --metadata-input-height: calc(var(--metadata-label-font-size) * 2.1);
}

.metadata-property-key .metadata-property-icon {
    --icon-color: #bdeec;
    --icon-size: 1rem;
}

.metadata-container {
    background-color: #fafafa;
    border-radius: 4px;
    padding: 12px;
}

```


### Bases

Bases 允许你将知识库中的任何一组笔记，瞬间转换成一个强大的、可视化的数据库。

#### 工作原理

Bases 的底层依然是本地 Markdown 文件，所有数据都源自笔记文件头部的 YAML 属性。

1. **`.base` 配置文件**：一种新文件，它不存储任何数据，只定义了如何展示数据，比如数据来源是哪个文件夹、用什么视图（表格、看板）、设置了哪些筛选和排序规则。
2. **数据源**：数据库的每一行，都对应一篇 Markdown 笔记。数据库的每一列，都对应笔记里的一个 YAML 属性。无缝衔接，零成本上手。
3. **强大的自定义能力**：可以创建不同的视图，设置复杂的筛选条件，甚至利用公式创造动态计算的属性。其灵活性足以让你为任何项目量身打造管理面板。

#### 创建

有以下几种方式可以创建 base 文件：
1. 命令窗口
2. 文件浏览器
3. Ribbon 中

#### 嵌入

1. 先创建 base 文件，然后使用 `![[File.base]]` 以链接进行嵌入
2. 直接以代码块形式嵌入：
```
	```base
	filters:
	  and:
		- file.hasTag("example")
	views:
	  - type: table
		name: Table
	```
```


#### 案例

1. [使用Base数据库追踪项目状态](https://forum-zh.obsidian.md/t/topic/53092)




## 第三方插件

[【小白教程】修改目录文件顺序，实现自定义排序/拖动排序 - 经验分享 - Obsidian 中文论坛](https://forum-zh.obsidian.md/t/topic/17849)

