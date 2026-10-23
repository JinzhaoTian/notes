React 起源于 Facebook 的内部项目，因为该公司对市场上所有 JavaScript MVC 框架，都不满意，就决定自己写一套，用来架设 Instagram 的网站，在2013年5月开源。

React 用来构建UI的 JavaScript 库，React 不是一个 MVC 框架，仅仅是视图（V）层的库。

特点：
1. 使用 JSX语法 创建组件，实现组件化开发，为函数式的 UI 编程方式打开了大门。
2. 性能高的让人称赞：通过 **Diff算法** 和 **虚拟DOM** 实现视图的高效更新。
3. HTML 仅仅是个开始。

> [!虚拟DOM]
React 将 DOM 抽象为虚拟 DOM ，虚拟 DOM 其实就是用一个对象来描述 DOM ，通过对比前后两个对象的差异，最终只把变化的部分重新渲染，提高渲染的效率。

> [!Diff算法]
> 当你使用 React 的时候，在某个时间点 render() 函数创建了一棵 React 元素树，在下一个 state 或者 props 更新的时候，render() 函数将创建一棵新的 React 元素树，React 将对比这两棵树的不同之处，计算出如何高效的更新 UI（只更新变化的地方）。


React 组件是继承 React.Component 类的ES6类，render() 是React组件唯一必需的方法，该方法的返回值是渲染到页面的内容。

```JavaScript
import React from 'react';

class ProductList extends React.Component {
    constructor(props) {
        super(props);
    }
    
    render() {
        return (
            <div className='ui unstackable items'>
                Hello, friend! I am a basic React component.
            </div>
        );
    }
}
```

# 底层原理

React 的核心原理通过 Virtual DOM、Diffing 算法、Fiber 架构、组件生命周期和 Hooks 等机制，提供了高效的 UI 更新和管理方式。这些原理使得 React 在处理大型和复杂的应用时能够保持高性能和高响应性。
## Virtual DOM

React 使用 Virtual DOM（虚拟 DOM）来优化页面更新的性能。Virtual DOM 是实际 DOM 的轻量级副本，它存在于内存中。React 元素被创建为 JavaScript 对象，代表了真实 DOM 中的节点。这些对象被存储在内存中，并通过比较新旧 Virtual DOM 的差异来高效地更新真实 DOM。

## Diffing Algorithm

React 使用一种高效的 Diffing 算法来比较新旧 Virtual DOM 树的差异。这个算法可以快速地确定哪些部分需要更新，并生成最小的更新补丁（patch）来应用到实际 DOM 中。这个过程叫做 "reconciliation"。

## Fiber Architecture

React 16 引入了 Fiber 架构，重新实现了协调（reconciliation）算法，以提高性能和响应能力。Fiber 是对 React 核心算法的一种重新实现，旨在使其能够分割渲染工作并在较长任务之间进行优先级调度。这样，React 可以更好地处理高优先级的更新（如用户输入），提高应用的响应速度。

## Component Lifecycle

React 组件有一组生命周期方法，这些方法允许开发者在组件的不同阶段执行特定的操作。生命周期方法分为三个阶段：
- 挂载阶段：组件被创建并插入到 DOM 中。
- 更新阶段：组件的状态或属性发生变化时触发。
- 卸载阶段：组件从 DOM 中移除时触发。

## Hooks

React 16.8 引入了 Hooks，它们允许在不编写类的情况下使用状态和其他 React 特性。Hooks 是一些特殊的函数，像 `useState`、`useEffect`，它们可以让你在函数组件中管理状态和副作用。

**在 Hooks 出现之前，只有类组件才能使用状态和生命周期方法**。函数组件相对简单，但无法处理复杂的状态逻辑和副作用。Hooks 的引入解决了这个问题，使得函数组件可以：

1. 使用状态（`useState`）
2. 管理副作用（`useEffect`）
3. 使用上下文（`useContext`）
4. 其他复杂的 React 功能


使用 Hooks 时需要遵循以下两个基本规则：

1. **只能在函数组件或自定义 Hook 中调用 Hooks**。不能在普通的 JavaScript 函数中调用 Hooks。
2. **只能在顶层调用 Hooks**。不要在循环、条件语句或嵌套函数中调用 Hooks。这是为了确保每次渲染时 Hooks 都以相同的顺序被调用，从而保证状态的一致性。

### 函数组件和类组件

在 React 中，组件可以用两种主要方式定义：函数组件和类组件。每种方式都有其独特的特性和适用场景。

何时使用：
- **函数组件**：适用于大多数场景，特别是当组件逻辑相对简单，或当你希望使用 Hooks 来管理状态和副作用时。
- **类组件**：在需要使用生命周期方法或处理更复杂的状态和逻辑时，可以考虑使用类组件。不过，随着 Hooks 的普及，类组件的使用频率有所下降。

#### 函数组件

函数组件是一个接受 props 作为参数**并返回 React 元素**的普通 JavaScript 函数。它是 React 中用于定义 UI 组件的一种方式，通常用于定义简单的、无状态的组件。

```js
import React, { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, [count]);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}

export default Counter;
```

- **简洁**：代码更加简洁，语法简单。
- **无状态**：传统上，函数组件是无状态的，但自从 React 16.8 引入 Hooks 后，函数组件也可以管理状态和副作用。
- **Hooks**：可以使用 Hooks（如 `useState` 和 `useEffect`）来处理状态和副作用。
- **无生命周期方法**：没有类组件的生命周期方法，但可以通过 Hooks 实现类似的效果。



#### 类组件

类组件是通过扩展 `React.Component` 类创建的，必须定义一个 `render` 方法来返回 React 元素。

```js
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={this.handleClick}>Click me</button>
      </div>
    );
  }
}

export default Counter;
```

- **状态管理**：可以通过 `this.state` 管理组件的内部状态。
- **生命周期方法**：提供了一系列生命周期方法（如 `componentDidMount`、`componentDidUpdate`、`componentWillUnmount`）来控制组件的各个阶段。
- **绑定上下文**：在类组件中使用 `this`，需要注意方法的上下文绑定问题。

### Props 和 State

Props 和 State 通常结合使用来构建复杂的组件和应用。父组件可以通过 props 传递数据和回调函数给子组件，子组件可以使用 state 来管理自身的动态数据，并通过回调函数通知父组件。
#### Props

Props（properties）是组件的输入参数，用于**将数据从父组件传递到子组件**。Props 是只读的，子组件不能直接修改它们。

- **只读**：子组件不能修改传递给它的 props。
- **单向数据流**：数据由父组件流向子组件，确保了数据的单向流动性，有助于维护应用的可预测性和调试性。
- **传递数据**：用于在组件之间传递数据，**甚至是函数**。
- **静态**：组件接收的 props 一旦被传递，不能在子组件内部改变。

#### State

State 是**组件内部**的数据管理机制，允许组件维护和更新其自身的状态。State 是可变的，当 state 发生变化时，**组件会重新渲染**以反映最新的状态。

- **可变**：State 是组件私有的，可以通过特定的方法（如 `setState` 或 `useState`）进行更新。
- **触发重新渲染**：当 state 发生变化时，组件会重新渲染，以反映最新的状态。
- **本地化**：state 仅在定义它的组件中可用，但可以通过 props 传递给子组件。
- **动态**：适用于需要频繁变化的数据，如用户输入、表单内容、计数器等。



### useState

`useState` 是一个 Hook，用于在函数组件中添加状态。它返回一个状态变量和一个更新该状态的函数。

```js
import React, { useState } from 'react';

function Counter() {
  // 声明一个叫做 "count" 的状态变量，初始值为 0
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

- `useState` 的参数是状态的初始值，可以是任何类型。
- `useState` 返回一个数组，第一个元素是当前状态值，第二个元素是一个函数，用于更新状态。
- 当调用更新函数时，React 会重新渲染组件并使用新的状态值。



### useEffect

`useEffect` 是一个 Hook，用于在函数组件中执行副作用（如数据获取、订阅或手动更改 DOM）。它相当于类组件中的生命周期方法 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 的组合。

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用浏览器 API 更新页面标题
    document.title = `You clicked ${count} times`;

    // 可选的清理函数，相当于 componentWillUnmount:
    return () => {
      // 这里执行清理操作，如取消订阅或清理定时器
      document.title = 'React App';
    };
  }, [count]); // 仅在 count 变化时执行副作用

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

- `useEffect` 接受两个参数：一个副作用函数和一个依赖数组。
    - 副作用函数在组件渲染**后**执行。
    - 依赖数组中的变量发生变化时，副作用函数会重新执行。如果没有依赖数组，每次组件渲染时都会执行副作用。
- 副作用函数可以返回一个可选的清理函数，该函数在组件卸载或在下一次执行副作用前执行。


### useLayoutEffect

`useLayoutEffect` 是在组件完成渲染之后、浏览器执行绘制之前同步触发的。这意味着在 DOM 更新之后，浏览器绘制之前，`useLayoutEffect` 中的函数会被同步调用。由于它是在绘制之前执行的，因此有可能阻塞组件的渲染过程。

`useEffect` 的副作用操作是在组件渲染完成后的”提交阶段”执行的。这意味着在浏览器完成绘制后，用户才能看到 `useEffect` 产生的结果。这种异步的特性使得它在处理如数据获取、订阅事件等需要等待的副作用操作时非常有用。

`useLayoutEffect` 的副作用操作是在组件渲染完成后的”布局阶段”执行的。由于它是在浏览器更新屏幕之前同步触发的，因此可以确保副作用的执行不会引起渲染跳跃，从而提供更流畅的用户体验。然而，如果 `useLayoutEffect` 中的操作非常耗时，那么可能会导致页面响应变慢，影响到用户的交互体验。



### useRef

`useRef` 是 React 的一个 Hook，用于创建一个可变的 ref 对象，其 `.current` 属性被初始化为传递的参数，并且可以在组件的整个生命周期内保持不变。`useRef` 最常见的用途是访问 DOM 元素或在不同的渲染之间保存可变值，而**无需触发组件重新渲染**。

```js
import React, { useRef } from 'react';

function TextInputWithFocusButton() {
  const inputEl = useRef(null);

  const onButtonClick = () => {
    inputEl.current.focus();
  };

  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```




## JSX

JSX 是一种 JavaScript 语法扩展，允许在 JavaScript 代码中编写类似 XML 的代码。JSX 代码在编译时被转换为 React.createElement 调用，创建对应的 React 元素。




# 库

### React-PDF

设计一个pdf展示页面，并进行高亮批注。

```js
import { theme } from "antd";

import React, { useEffect, useState, useRef } from "react";
import { Document, Page, pdfjs } from "react-pdf";
import "react-pdf/dist/Page/AnnotationLayer.css";
import "react-pdf/dist/Page/TextLayer.css";

pdfjs.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjs.version}/pdf.worker.mjs`;

const HIGHLIGHT_STYLE = {
    DEFAULT: "rgba(255, 255, 0, 0.12)",
    HOVER: "rgba(255, 120, 0, 0.12)",
};

export default function ResultPdfComponent({ props }) {
    const {
        token: { colorBgContainer, borderRadiusLG },
    } = theme.useToken();

    const pdfCanvasRef = useRef(null);
    const highlightCanvasRef = useRef(null);
    const containerRef = useRef(null);
    const [pageRendered, setPageRendered] = useState(false);

    const drawHighlightArea = (context, highlight) =>{
        context.fillRect(highlight.x, highlight.y, highlight.width, highlight.height);
    };

    const highlightArea = (highlights, selectedHighlightIndex = null) => {
        const highlightCanvas = highlightCanvasRef.current;
        if (highlightCanvas) {
            const context = highlightCanvas.getContext("2d");
            context.clearRect(0, 0, highlightCanvas.width, highlightCanvas.height);

            highlights.forEach((highlight, index) => {
                context.fillStyle = index === selectedHighlightIndex ? HIGHLIGHT_STYLE.HOVER: HIGHLIGHT_STYLE.DEFAULT;
                drawHighlightArea(context, highlight);
            });
        }
    };

    const scrollToHighlight = (selectedHighlight) => {
        const highlightCanvas = highlightCanvasRef.current;
        const container = containerRef.current;
        if (highlightCanvas && container) {
            const { x, y, width, height } = props.highlights[selectedHighlight];
            const canvasRect = highlightCanvas.getBoundingClientRect();
            const containerRect = container.getBoundingClientRect();

            const highlightCenterY = y + height / 2;
            const containerCenterY = container.clientHeight / 2;
            const scrollY = container.scrollTop + (canvasRect.top - containerRect.top) + highlightCenterY - containerCenterY;

            const highlightCenterX = x + width / 2;
            const containerCenterX = container.clientWidth / 2;
            const scrollX = container.scrollLeft + (canvasRect.left - containerRect.left) + highlightCenterX - containerCenterX;

            container.scrollTo({
                top: scrollY,
                left: scrollX,
                behavior: "smooth",
            });
        }
    };

    useEffect(() => {
        if (pageRendered) {
            highlightArea(props.highlights);
        }
    }, [props.highlights, pageRendered]);

    useEffect(() => {
        if (pageRendered && props.selectedHighlight !== null) {
            highlightArea(props.highlights, props.selectedHighlight);
            scrollToHighlight(props.selectedHighlight);
        }
    }, [props.selectedHighlight, pageRendered]);

    const onRenderSuccess = (page) => {
        const pdfCanvas = pdfCanvasRef.current;
        const highlightCanvas = highlightCanvasRef.current;
        if (pdfCanvas && highlightCanvas) {
            highlightCanvas.width = pdfCanvas.width;
            highlightCanvas.height = pdfCanvas.height;
        }
        setPageRendered(true);
    };

    return (
        <div
            ref={containerRef}
            style={{
                padding: 24,
                width: "100%",
                height: 760,
                minHeight: 600,
                minWidth: 200,
                background: colorBgContainer,
                borderRadius: borderRadiusLG,
                overflow: "auto",
                position: "relative",
            }}
        >
            <Document file={props.file}>
                <Page pageNumber={1} canvasRef={pdfCanvasRef} onRenderSuccess={onRenderSuccess} />
            </Document>
            <canvas ref={highlightCanvasRef} style={{ position: "absolute", top: 24, left: 24, pointerEvents: "none" }} />
        </div>
    );
}

```



### React-Draggable

设计一个可拖动容器

```js
import Draggable from "react-draggable";

const handleResize = (key, deltaX) => {
	setColSizes((prevSizes) => {
		const totalFlex = Object.values(prevSizes).reduce((a, b) => a + b, 0);
		const newSizes = { ...prevSizes };
		const delta = deltaX / 200; // Adjust deltaX based on the actual layout width

		if (key === "col1") {
			newSizes.col1 += delta;
			newSizes.col2 -= delta;
		} else if (key === "col2") {
			newSizes.col2 += delta;
			newSizes.col3 -= delta;
		} else if (key === "col3") {
			newSizes.col3 += delta;
			newSizes.col4 -= delta;
		}

		// Ensure no column is less than 1 flex unit
		Object.keys(newSizes).forEach((col) => {
			if (newSizes[col] < 1) {
				if (col === "col4") {
					newSizes[col] = 0;
				} else {
					newSizes[col] = 1;
				}
			}
		});

		return newSizes;
	});
};



<div style={{ flex: colSizes.col1, padding: 5, position: "relative" }}>
	<ResultIssuesComponent onSelect={onSelect} />
	<Draggable
		axis="x"
		onDrag={(e, data) => handleResize("col1", data.deltaX)}
		position={{ x: 0, y: 0 }}
>
		<div
			style={{
				width: "5px",
				cursor: "col-resize",
				height: "100%",
				position: "absolute",
				top: 0,
				right: 0,
			}}
		/>
	</Draggable>
</div>
```


# 项目开发


## 文件夹结构


创建后，你的项目应如下所示：

```
frontend/
	├── node_modules/
	├── public/
	│ ├── index.html
	│ └── favicon.ico
	├── src/
	│ ├── App.css
    │ ├── App.js
    │ ├── App.test.js
    │ ├── index.css
    │ ├── index.js
	│ └── logo.svg
	├── package.json
	└── README.md
```

 对于要构建的项目，这些文件必须以确切的文件名存在：`public/index.html` 是页面模板; `src/index.js` 是 JavaScript 入口点。你可以删除或重命名其他文件。

可以在 `src` 中创建子目录。 为了加快重新构建的速度，Webpack 只处理 `src` 中的文件。 你需要**将任何 JS 和 CSS 文件放在 `src` 中**，否则 Webpack 将发现不了它们。

只能在 `public/index.html` 中使用 `public` 中的文件。 但是，你可以创建其他你想要的顶级目录，但是它们不会包含在生产版本中，因此你可以将它们用于文档等内容。

如果你安装了 Git 并且你的项目不是更大的存储库的一部分，那么将初始化一个新的存储库，从而产生一个额外的顶级 `.git` 目录。

