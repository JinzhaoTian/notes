[Tailwind CSS](https://www.tailwindcss.cn/) 是一个开源 CSS 框架，其工作原理是扫描所有 HTML 文件、JavaScript 组件以及任何模板中的 CSS 类（class）名，然后生成相应的样式代码并写入 到一个静态 CSS 文件中。

与传统的 CSS 框架不同，Tailwind 不提供预定义的组件，而是提供了一组低级的工具类，开发者可以使用这些工具类来设计自己的组件。

**主要特点**：
1. **功能类优先**：使用功能类（如 `bg-blue-500`、`text-center`）直接在 HTML 中设置样式，避免了样式重复和样式冲突。
2. **响应式设计**：Tailwind 支持响应式设计，可以根据不同的屏幕尺寸应用不同的样式。
3. **定制化**：Tailwind 可以通过配置文件进行深度定制，开发者可以轻松修改默认主题、添加自定义颜色、间距等。
4. **快捷构建**：通过组合多个类名，开发者可以快速构建复杂的布局和组件。
5. **不再需要 CSS 文件**：在很多情况下，使用 Tailwind 可以减少对额外 CSS 文件的需求。


## 运行原理

1. **配置**：Tailwind 使用一个配置文件（通常是 `tailwind.config.js`），开发者可以在其中定义主题、颜色、字体、间距等自定义属性，这个配置文件会影响生成的 CSS。
```js
// tailwind.config.js

module.exports = {
  purge: ['./src/**/*.html', './src/**/*.js'],   // 启用 PurgeCSS
  theme: {
    extend: {
      colors: {
        customColor: '#123456',
      },
    },
  },
  variants: {},
  plugins: [],
};
```

2. **生成**：在**构建过程**中，Tailwind 会根据配置文件生成**大量的** CSS 类。这些类对应不同的样式，例如背景色、边框、字体大小、间距等。这些类通常采用特定的命名约定，比如 `bg-red-500` 或 `p-4`。
	-  **按需生成**： 在开发过程中，使用 PostCSS 和 PurgeCSS，Tailwind 可以在构建阶段只提取实际使用的类，从而减小最终生成的 CSS 文件的大小。这样可以确保只包含实际使用的样式，避免了冗余。
```js
// postcss.config.js

module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

3. **使用**：开发者在 HTML 或 JSX 中使用这些类名来直接应用样式。
4. **响应式**：Tailwind 允许开发者通过特定的类名模式轻松实现响应式设计和状态变体（如悬停、焦点等）。例如，`md:bg-green-500` 只在中等屏幕及以上时应用绿色背景。

## 语法

Tailwind 的重点是提供实用的工具类（Utility Classes）来快速应用样式，而不是直接使用传统的 CSS 选择器。以下是 Tailwind CSS 中常见的一些语法和实现方式。

### **伪类选择器**

Tailwind CSS 提供了一些针对伪类的快捷类，可以方便地为元素添加特定的状态样式。

- **`:hover`**：通过 `hover:` 前缀来为元素添加悬停状态样式：
```html
<button class="bg-blue-500 hover:bg-blue-700">Hover me!</button>
```

- **`:focus`**：通过 `focus:` 前缀来为元素添加焦点状态样式：
    ```html
<input class="focus:outline-none focus:ring-2 focus:ring-blue-500" />
```

- **`:active`**：通过 `active:` 前缀为元素添加激活状态样式：
```html
<button class="bg-blue-500 active:bg-blue-700">Click me!</button>
```
    
- **`:disabled`**：使用 `disabled:` 前缀为禁用元素设置样式：
```html
<button class="disabled:bg-gray-400" disabled>Disabled Button</button>
```

- **`:checked`**：为选中状态的元素添加样式：
```html
<input type="checkbox" class="checked:bg-blue-500" />
```

### **状态和变体（State Variants）**

Tailwind 还允许使用状态变体来处理响应式设计和动态交互。

- **响应式设计（Responsive Design）**：Tailwind 使用不同的屏幕大小前缀（如 `sm:`, `md:`, `lg:`）来应用响应式样式。
```html
<div class="text-center md:text-left">
  This text is centered on small screens and left-aligned on medium screens and above.
</div>
```

- **`group` 组合类**：用于组合父子元素的样式。例如，通过父级 `group` 类控制子元素的样式：
```html
<div class="group">
  <button class="group-hover:bg-green-500">Hover me</button>
</div>
```



### 特殊语法

`[&_svg]:size-4` ：
- **`&`**：在 Tailwind CSS 中，`&` 用作父元素的占位符。它代表当前选择器所在的元素，在这里表示包含 `<svg>` 元素的父元素。
- **`_svg`**：这个语法是对父元素下的 `<svg>` 元素的直接选择。使用 `_` 连接符来明确指定元素类型。`_svg` 表示选中当前元素中的 `<svg>` 元素。
- **`:size-4`**：这是 Tailwind 的工具类，表示设置元素的大小。`size-4` 通常等于 `1rem` 或 `16px`，根据 Tailwind 配置的默认单位设置。

`[&>svg]:size-6` ：
- **`> svg`**：这个部分表示 "直接子元素" 的选择器。它选择当前元素下的所有 `<svg>` 元素，但只限于直接子元素（即不选择嵌套的 `<svg>`）。