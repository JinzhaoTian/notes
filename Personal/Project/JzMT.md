---
project: JzMT
tags:
  - 个人项目
  - 笔记
  - JzMT
priority: 3
---
文档编辑器根据编辑模式的不同可以分为纯文本编辑器、富文本编辑器、Markdown编辑器等等。其中，Markdown编辑器根据渲染模式的不同又可分为实时渲染模式、所见即所得（What You See Is What You Get，WYSIWYG）模式。

## 需求分析

目标是做一个类似于 Obsidian 的 Web 端 Markdown 笔记系统，其中比较可取的是 Markdown 编辑提示，以及流畅的编辑体验，此外就是有强大的双向链接和图谱视图功能（毕竟是 Obsidian 的第一开发理念）。

### 需求调研

[How can I implement live preview feature on my markdown editor (that I am implementing ) in the same window like Typora does (without having a separate pane like stackedit)? : r/webdev](https://www.reddit.com/r/webdev/comments/fr5u7m/how_can_i_implement_live_preview_feature_on_my/)

[A Typora-like editing mode (edit and preview at the same time) - Feature archive - Obsidian Forum](https://forum.obsidian.md/t/a-typora-like-editing-mode-edit-and-preview-at-the-same-time/1953/69?page=2)
[brrd/abricotine: Markdown editor with inline preview](https://github.com/brrd/Abricotine/?tab=readme-ov-file)

[How to build a Markdown editor with real-time preview](https://zuunote.com/blog/how-to-build-a-markdown-editor-with-real-time-editing/)


### 主要功能

#### Live Inline Preview

就是 Typora 模式，我觉得这个是最重要的，是Markdown最优雅的输入方式。

#### 文档管理

- project
	- doc
- TODOs
- Tags
- 编辑历史？
- user contribution


侧边栏：
- 简略布局
- 卡片布局


[AppIcon Forge - Create & Customize Stunning App Icons Online](https://zhangyu1818.github.io/appicon-forge/)



#### 文件导出

- markdown 格式
- pdf 格式
- txt 格式
- 富文本 格式


### Markdown 支持

1. 基础语法及渲染支持，上下文感知与动态补全。
2. 即时 Markdown 解析：当用户在编辑器中输入 Markdown 符号时，要实时解析内容并更新渲染。
	- 根据 Markdown 规则，将语法解析为 HTML 元素，但仅在渲染层次进行处理，不改变原始文本。
	- 不移除 Markdown 格式符号，而是通过样式和交互手段让符号与渲染效果共存。
3. 富文本与纯文本同步显示：使用了**装饰器**的机制，实现**保留格式符号并同步渲染**的效果。
	- 文本分层渲染：Markdown 格式符号（如 `#`、`**` ）仍然保留在底层文本流中，用户在光标移动和编辑时可以直接修改这些符号。在渲染层，应用额外的 CSS 样式或虚拟 DOM 节点覆盖来呈现最终的效果。
	- 光标位置校正：实现了对光标的智能处理，避免由于格式符号渲染样式的变化导致光标位置错误。例如，当用户在 `**粗体**` 中间插入字符时，光标能准确定位。
	- **动态样式应用**：使用 CSS 类动态更新符号的样式。
```css
.obsidian-bold {
    font-weight: bold;
}
.markdown-syntax {
    color: transparent; /* 隐藏但保留占位 */
}
```



### 双向链接



### 协同编辑？






### （2025.11.10）与 Github 协同

我在微软的过往的项目经验中，有一个基于 Github 的平台开发 OpenAPI Hub，现在可以基于这个经验，与这个笔记软件一起开发一个，将数据存储在 Github 中，然后 Web 托管一个笔记服务，可以打开 Github 中的项目，可以先做笔记的，然后还可以添加code的，最后可以上线在线编辑功能。



## 技术选型

### 编辑器

1. [Tiptap](https://tiptap.dev/docs) ：支持富文本和 Markdown，基于 ProseMirror，易于集成 Tailwind CSS，提供丰富的扩展（如表格、代码块、Markdown 支持等）。
   
   但是官方说不太支持 Markdown，![](imgs/Pasted%20image%2020241116130335.png)如果是要以 Markdown 文本作为输入输出格式，建议用别的。

2. [CodeMirror](../../Language/Node/前端开发/CodeMirror.md) ：注重 Markdown 语法高亮，适合轻量级需求，可以通过 Tailwind CSS 定制编辑器的外观。
3. [Lexical](https://lexical.dev/docs/intro) ：性能高、架构现代化，适合构建复杂 Markdown 编辑器。内置支持 Tailwind CSS，通过插件化架构添加 Markdown 支持。![](../../Language/Node/前端开发/imgs/Pasted%20image%2020241119174811.png)


2. [七款优质开源项目，让你的Markdown编辑更加行云流水 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/178981578)
3. [Etherpad](https://etherpad.org/)，Code：[ether/etherpad-lite: Etherpad: A modern really-real-time collaborative document editor. (github.com)](https://github.com/ether/etherpad-lite)
4. [sytone/obsidian-remote: Run Obsidian.md in a browser via a docker container. (github.com)](https://github.com/sytone/obsidian-remote)


### 存储方式

**方案一：单字段存储（最简单）**

```sql
CREATE TABLE notes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL, -- 存储原始 Markdown 文本
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    user_id INT, -- 关联用户
    notebook_id INT -- 关联笔记本（可选）
    -- ... 其他元数据字段 (标签、状态等)
);
```
- **优点:**
    - **极其简单：** 实现快速，结构清晰。
    - **完整保留：** 原样存储 Markdown，保证无损。
    - **写入快：** 只需插入/更新一个文本字段。
- **缺点:**
    - **查询效率低：** 搜索笔记内容（`WHERE content LIKE '%keyword%'`）效率极差，尤其是大数据量时。
    - **无法结构化查询：** 难以直接查询 Markdown 中的结构化元素（如标题级别、链接目标、代码块语言）。
    - **渲染依赖应用层：** 每次展示都需要应用层将 Markdown 转换为 HTML/其他格式。


**方案二：分离内容与元数据 + 搜索索引（推荐）**

```sql
CREATE TABLE notes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL, -- 存储原始 Markdown 文本
    content_html TEXT, -- **可选：** 存储渲染后的 HTML (加速读取)
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    user_id INT,
    notebook_id INT
    -- ... 其他元数据字段
);

-- **核心改进：** 为高效搜索建立专门的索引表
CREATE TABLE note_search_index (
    note_id INT NOT NULL,
    search_text TEXT NOT NULL, -- 包含去 Markdown 符号的纯文本、标题、标签等
    -- ... 可添加其他用于搜索的字段（如分词后的词向量）
    FULLTEXT INDEX (search_text) -- 使用数据库的全文索引 (MySQL, PostgreSQL)
    -- 或者使用专门的搜索引擎 (Elasticsearch, Typesense, Meilisearch)
);
```
- **优点:**
    - **保留原始数据：** `content` 字段完整存储 Markdown。
    - **高效搜索：** 利用全文索引或专用搜索引擎在 `search_text` 上实现快速、强大的关键字、短语搜索。
    - **优化读取：** `content_html` 字段（可选）避免了每次请求都进行 Markdown 渲染，极大提升读取速度（牺牲存储空间和写入时间）。
    - **结构清晰：** 元数据和内容分离，易于管理。
- **缺点:**
    - **复杂度增加：** 需要维护搜索索引（写入笔记时需要更新 `search_text`，可能还需更新 `content_html`）。
    - **存储开销：** `content_html` 和 `search_text` 会增加存储空间。
    - **索引更新延迟：** 全文索引/搜索引擎可能有短暂延迟。


**方案三：结构化存储（最复杂）**

将 Markdown 文档解析成抽象语法树（AST），然后将 AST 的节点（段落、标题、列表项、代码块、链接等）及其属性（级别、语言、URL）存储在数据库的关联表中。

```sql
CREATE TABLE notes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    user_id INT,
    notebook_id INT
    -- ... 元数据
);

CREATE TABLE note_blocks (
    id INT PRIMARY KEY AUTO_INCREMENT,
    note_id INT NOT NULL,
    block_type ENUM('paragraph', 'heading', 'list', 'list_item', 'code', 'blockquote', 'thematic_break', 'image', 'link') NOT NULL,
    parent_block_id INT, -- 用于嵌套结构（如列表项）
    ordering INT NOT NULL, -- 块在文档中的顺序
    raw_text TEXT, -- 该块内的原始文本（对于纯文本块）
    heading_level TINYINT, -- 仅对 heading 类型有效
    code_language VARCHAR(50), -- 仅对 code 类型有效
    link_url VARCHAR(2048), -- 仅对 link/image 类型有效
    alt_text VARCHAR(255), -- 仅对 image 类型有效
    -- ... 其他块特定属性
    FOREIGN KEY (note_id) REFERENCES notes(id) ON DELETE CASCADE,
    FOREIGN KEY (parent_block_id) REFERENCES note_blocks(id) ON DELETE CASCADE
);
```
- **优点:**
    - **极致灵活查询：** 可以执行非常精细的查询（“查找所有包含 Python 代码块的笔记”、“查找所有 H2 标题为 'TODO' 的笔记”、“查找所有指向 example.com 的链接”）。
    - **精准修改：** 可以只更新文档中的某个特定块。
    - **富客户端交互：** 非常适合实现类似 Notion 的块级编辑和实时协作。
    - **独立渲染：** 客户端可以根据块数据自由渲染（支持多种视图）。
- **缺点:**
    - **极端复杂：** 设计、实现、维护难度很高。需要强大的 Markdown 解析器（服务端或客户端）。
    - **写入开销大：** 每次保存笔记都需要完整解析 Markdown 并插入/更新大量关联记录。
    - **读取开销大：** 重建完整 Markdown 文档或 HTML 需要查询并组装多个块，可能较慢。
    - **存储开销：** 存储大量小记录，元数据占比高。
    - **潜在不一致：** 解析和存储过程出错可能导致数据不一致。

#### 元数据管理

- **无论如何存储内容，将笔记的元数据（标题、创建/修改时间、所属用户/笔记本、标签、状态、收藏状态等）存储在关系型数据库的标准字段中是标准做法。** 这些字段的查询（按标签过滤、按时间排序）非常高效。
- **标签系统：** 通常需要多对多关系，设计 `tags` 表和 `note_tags` 关联表。

#### 版本控制

- 如果需要保存历史版本，**不要覆盖原内容**！单独设计 `note_versions` 表，结构可以类似主表（如方案二），包含 `note_id`, `version_number`, `content` (Markdown), `content_html` (可选), `saved_at`, `saved_by` 等字段。每次编辑保存时插入新版本记录。


#### 数据库选择

- **关系型数据库（PostgreSQL, MySQL/MariaDB）：** 对方案一、二和三都适用。PostgreSQL 的全文检索（`tsvector`）通常比 MySQL 的更强大，JSONB 类型对存储部分结构化元数据也有优势。
- **文档数据库（MongoDB）：** 天然适合存储方案一（整个文档作为一个 BSON 文档）和方案二（可以将元数据和 `content` 放在一个文档，`content_html` 和搜索索引可能需要额外处理或结合其他技术）。对方案三（结构化块）也可以实现，但关系型可能更擅长处理块之间的顺序和嵌套关系。

**关联表：**
- `tags` 表 + `note_tags` 关联表 (多对多)。
- `note_versions` 表 (如果需要历史版本)。
- `attachments` 表 (存储附件元数据和指向对象存储的 URL)。



#### 附件存储

- 笔记中引用的图片等附件，**强烈建议存储在对象存储服务（如 AWS S3, MinIO, Cloudinary）或专门的文件系统**。
- 在数据库（`content` 字段或结构化存储的块中）只存储附件的**URL引用**。避免将二进制大文件直接存入数据库。


#### 搜索需求

- 如果搜索是核心功能（用户需要快速找到笔记内容），**方案二（分离+搜索索引）是必须的**。单字段存储无法满足性能要求。
- 考虑使用数据库内置全文索引（MySQL FULLTEXT, PostgreSQL tsvector）或集成更强大的专用搜索引擎（Elasticsearch, Typesense, Meilisearch）。