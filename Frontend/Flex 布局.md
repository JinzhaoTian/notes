Flex布局，全称为Flexbox（Flexible Box），是一种CSS布局模式，专为在一维空间内（即一行或一列）排列元素而设计。它非常适合用来在横向或纵向的容器中对元素进行对齐、分布和空间分配，使得布局更加灵活、适应性更强。

### 基本概念

1. **容器（Container）**: 包含子元素的父级容器，设置`display: flex;`或`display: inline-flex;`将其转换为Flex容器。
2. **项目（Item）**: Flex容器内的每个子元素称为项目，它们会自动根据容器的设置而排列和对齐。

### 主要属性

#### 容器属性

- `flex-direction`: 定义主轴方向，项目会沿主轴排列。
    - `row`: 水平，从左到右排列（默认）。
    - `row-reverse`: 水平，从右到左排列。
    - `column`: 垂直，从上到下排列。
    - `column-reverse`: 垂直，从下到上排列。
- `justify-content`: 定义项目沿主轴的对齐方式。
    - `flex-start`: 从起点开始对齐。
    - `flex-end`: 从终点开始对齐。
    - `center`: 居中对齐。
    - `space-between`: 项目之间平均分布，首尾不留空隙。
    - `space-around`: 项目之间及首尾留有相等的空间。
    - `space-evenly`: 项目和边界的空间相等。
- `align-items`: 定义项目沿交叉轴的对齐方式。
    - `flex-start`: 从交叉轴的起点对齐。
    - `flex-end`: 从交叉轴的终点对齐。
    - `center`: 在交叉轴上居中。
    - `baseline`: 按项目的文本基线对齐。
    - `stretch`: 如果项目未设置高度或宽度，则填充容器。
- `flex-wrap`: 决定项目是否换行。
    - `nowrap`: 不换行（默认）。
    - `wrap`: 换行。
    - `wrap-reverse`: 换行并颠倒行的顺序。

#### 项目属性

- `flex-grow`: 定义项目的放大比例，数值越大，该项目占据的空间越多。
- `flex-shrink`: 定义项目的缩小比例，数值越大，该项目在空间不足时会缩小得更多。
- `flex-basis`: 定义项目的初始大小，可设定固定值或自动计算。
- `align-self`: 允许项目单独设置在交叉轴上的对齐方式，覆盖`align-items`设置。

### 使用示例

```css
.container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}
.item {
  flex: 1; /* 等价于flex-grow: 1, flex-shrink: 1, flex-basis: 0% */
}
```

```html
<div class="container">
  <div class="item">1</div>
  <div class="item">2</div>
  <div class="item">3</div>
</div>
```

	在这个例子中，`.container`是一个Flex容器，`flex-direction: row`让项目在水平方向排列，`justify-content: center`使项目在主轴上居中对齐，`align-items: center`让项目在交叉轴上居中对齐。