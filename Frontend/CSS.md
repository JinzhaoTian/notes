## CSS 选择器

CSS 选择器用于选中 HTML 文档中的元素，并为其应用样式。根据选择范围和条件的不同，CSS 选择器可以分为不同类型。

**CSS选择器优先级** ：
- 内联样式：优先级最高。
- ID 选择器：次高。
- 类选择器、伪类选择器和属性选择器：优先级较低。
- 元素选择器：优先级最低。
- 通配符选择器 `*` 和伪元素选择器 `::before`/`::after` 没有太高优先级。

**CSS选择器的其他特性** ：
- **`:is()` 和 `:where()`**：用于减少重复代码和提高选择器的灵活性。
- **`:scope`**：选中文档中的作用域元素。

通过这些选择器，您可以非常灵活地为 HTML 元素应用样式，从而创建精美且响应式的网页。

### **基础选择器**

- **元素选择器（Type Selector）**： 选择指定类型的 HTML 元素。
```css
p {
  color: blue;
}
```

- **类选择器（Class Selector）**： 选择具有特定类名的元素。类选择器使用 `.` 开头。
```css
.container {
  padding: 20px;
}
```

- **ID 选择器（ID Selector）**： 选择具有特定 ID 的元素。ID 选择器使用 `#` 开头。
```css
#header {
  font-size: 24px;
}
```

- **通配符选择器（Universal Selector）**： 选择所有元素。
```css
* {
  margin: 0;
}
```
### **组合选择器**

- **后代选择器（Descendant Selector）**： 选中某个元素内的所有指定后代元素。
```css
div p {
  color: red;
}
```

- **子元素选择器（Child Selector）**： 选中某个元素的直接子元素。
```css
div > p {
  color: green;
}
```

- **相邻兄弟选择器（Adjacent Sibling Selector）**： 选中紧接在另一个元素后面的元素。
```css
h1 + p {
  color: yellow;
}
```

- **一般兄弟选择器（General Sibling Selector）**： 选中与指定元素具有相同父元素的所有兄弟元素。
```css
h1 ~ p {
  color: purple;
}
```

### **属性选择器**

属性选择器允许你根据元素的属性值进行选择。

- **基本属性选择器**：
```css
input[type="text"] {
  border: 1px solid black;
}
```

- **存在属性选择器**： 选中具有指定属性的元素。
```css
input[type] {
  background-color: lightblue;
}
```

- **属性值部分匹配**：
- **`^=`**：属性值以某个字符串开头。
```css

a[href^="https"] {
  color: green;
}
```

 - **`$=`**：属性值以某个字符串结尾。
```css
a[href$=".jpg"] {
  border: 2px solid red;
}
```

- **`*=`**：属性值包含某个子字符串。
```css
a[href*="example"] {
  text-decoration: underline;
}
```

### **伪类选择器**

伪类选择器用于选择特定状态的元素。常见的伪类包括：

- **`:hover`**：当鼠标悬停在元素上时。
```css
button:hover {
  background-color: blue;
}
```

- **`:first-child`**：选中父元素的第一个子元素。
```css
p:first-child {
  font-weight: bold;
}
```

- **`:last-child`**：选中父元素的最后一个子元素。
```css
p:last-child {
  margin-bottom: 0;
}
```

- **`:nth-child()`**：选中父元素中某个位置的子元素（可用数字或公式）。
```css
li:nth-child(2) {
  color: red;
}
```

- **`:not()`**：选中不符合条件的元素。
```css
div:not(.active) {
  opacity: 0.5;
}
```

### **伪元素选择器**

伪元素选择器用于选中特定的部分内容，通常与 DOM 元素的某些部分交互：

- **`::before`**：在元素内容前插入内容。
```css
h1::before {
  content: "★ ";
}
```

- **`::after`**：在元素内容后插入内容。
```css
p::after {
  content: "✔";
}
```

### **组合和复杂选择器**

你可以结合多个选择器来实现更复杂的样式规则：

- **`:root`**：表示文档的根元素，通常用于设置 CSS 变量。
```css
:root {
  --main-color: #ff5733;
}
```

- **`[attribute|="value"]`**：选中属性值以 `value` 开头，并允许以 `-` 分隔的值匹配。
```css
[lang|="en"] {
  font-family: Arial, sans-serif;
}
```



## 伪类

### `:focus`

表示获得焦点的元素，当用户点击或轻触一个元素或使用键盘的 Tab 键选择它时，它会被触发。

### `:hover`

在用户使用指针设备与元素进行交互时匹配，但不一定激活它。通常情况下，用户将光标（鼠标指针）悬停在元素上时触发。

