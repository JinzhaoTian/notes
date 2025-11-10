DOM（Document Object Model，文档对象模型）是浏览器用来表示和操作 HTML 和 XML 文档的编程接口，它以树状结构组织文档的内容。每个 HTML 元素都是树中的一个节点，开发者可以通过 JavaScript 操作这些节点，来动态更新页面内容和样式。

**DOM 的特性**
1. **结构化表示** ：DOM 是一种树结构，根节点是 `<html>`，每个元素和属性都是一个节点。
2. **实时性** ：DOM 是页面的直接表示，所有对 DOM 的操作都会实时反映到浏览器界面。
3. **性能问题：**
    - 直接操作 DOM 会触发重绘（Repaint）或重排（Reflow），影响性能。
    - DOM 操作会跨越 JavaScript 引擎和浏览器渲染引擎的边界（多次 DOM 操作会导致频繁的通信）。

```html
<div id="content">Hello, World!</div>
<script>
  // 使用 JavaScript 操作 DOM
  const div = document.getElementById("content");
  div.textContent = "Hello, DOM!";
</script>
```



## 虚拟 DOM

虚拟 DOM（Virtual DOM）是一种抽象层，常见于现代前端框架（如 React、Vue）。它通过在内存中维护一棵虚拟的 DOM 树，避免直接操作真实的 DOM。对页面的修改会先应用到虚拟 DOM 树中，再通过“差异计算”（diffing）找到最小的变更集，最后将变更应用到真实 DOM。

**虚拟 DOM 的特性**
1. **轻量级表示** ：虚拟 DOM 是真实 DOM 的抽象表示，通常是一个 JavaScript 对象，描述了 DOM 结构。
2. **高效更新** ：虚拟 DOM 通过 diff 算法比较前后两棵虚拟 DOM 树的差异，只对真实 DOM 应用必要的更改。
3. **框架依赖** ：虚拟 DOM 不是浏览器内置的，而是由前端框架（如 React）实现。


**虚拟 DOM 工作流程**
1. **初始化** ：根据组件的状态生成初始的虚拟 DOM 树。
2. **状态更新** ：状态或数据变化时，生成新的虚拟 DOM 树。
3. **Diffing 和 Patch** ：比较新旧虚拟 DOM 树，生成最小的差异（Patch）。
4. **更新真实 DOM** ：根据差异只修改需要更新的 DOM 节点。

**虚拟 DOM 的优势**
- 减少直接操作真实 DOM 的次数。
- 优化了多次操作 DOM 时的性能问题。
- 提供跨平台支持（如 React Native 将虚拟 DOM 映射到原生控件）。


```jsx
import React from "react";
import ReactDOM from "react-dom";

function App() {
  return <div>Hello, Virtual DOM!</div>;
}

ReactDOM.render(<App />, document.getElementById("root"));
```

