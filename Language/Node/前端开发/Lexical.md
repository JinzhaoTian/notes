[Lexical](https://lexical.dev/docs/intro)  是由 Meta 开发的一个现代化、高性能的文本编辑器框架，开发背景源于对现有文本编辑技术的局限性的认识，如 ContentEditable API 存在多样化的兼容性问题和性能瓶颈。为了解决现有文本编辑工具中的通用问题如性能低下、难以扩展、复杂的交互逻辑处理等，Meta 创造了 Lexical，不依赖于标准的 ContentEditable 特性，而是基于自己的架构构建，允许开发者更好地控制编辑器的行为和性能。

> [!TIP]
> 设计思路参考了 `ProseMirror` .


**核心特性** ：
1. **高性能**： Lexical 在设计时就将性能视为核心目标之一，通过智能的差异渲染算法和优化的数据结构，Lexical 提供了极为流畅的用户体验。它能够快速响应用户输入，即使是在处理大量文本或复杂格式时也不会出现延迟。
2. **可扩展性**： Lexical 提供了一个灵活的插件系统，允许开发者按照自己的需求添加新的功能。这包括但不限于自定义键绑定、文本样式、组件插入等特性。这种设计使得 Lexical 不仅限于富文本编辑，甚至可以扩展到代码编辑器等更多场景。
3. **易于集成**： 尽管 Lexical 具有复杂的内部实现，但它提供了简洁的 API 和丰富的文档，使得集成过程相对简单。它支持 React 框架，并通过提供钩子和组件来与 React 应用无缝集成，同时也支持非 React 环境的集成。
4. **功能丰富**： 除了基本的文本编辑功能外，Lexical 还内建了Markdown、撤销/重做功能、拼写检查等高级功能。允许开发者以插件的形式扩展其功能，以适应不同的使用场景。

## 原理

![](../../../Frontend/imgs/Pasted%20image%2020241119101513.png)


![](../../../Frontend/imgs/Pasted%20image%2020241119101604.png)

### 核心概念

1. **编辑器实例（Editor Instance）**：编辑器实例是 Lexical 编辑器的主要接口，它是编辑环境的核心，负责管理编辑器的状态和提供编辑操作的 API ，每个编辑器实例都独立管理其自己的文档状态。
```js
import {createEditor} from 'lexical';

const config = {
  namespace: 'MyEditor',
  theme: {
    ...
  },
  onError: console.error
};

const editor = createEditor(config);

const contentEditableElement = document.getElementById('editor');

editor.setRootElement(contentEditableElement);
```

2. **编辑器状态（Editor State）**：编辑器状态是一个不可变的数据结构，代表编辑器在特定时间点的完整状态，包括所有文本节点、选择范围、历史记录等信息。状态的不可变性可以方便地实现功能如撤销/重做。
```js
const stringifiedEditorState = JSON.stringify(editor.getEditorState().toJSON());

const newEditorState = editor.parseEditorState(stringifiedEditorState);
```


3. **编辑器更新（Editor Update）**：编辑器更新是对当前编辑器状态的一系列变更，这些变更在更新函数中执行，并且可以一起应用到状态中，确保变更的原子性。
```js
import {$getRoot, $getSelection, $createParagraphNode, $createTextNode} from 'lexical';

// Inside the `editor.update` you can use special $ prefixed helper functions.
// These functions cannot be used outside the closure, and will error if you try.
// (If you're familiar with React, you can imagine these to be a bit like using a hook
// outside of a React function component).
editor.update(() => {
  // Get the RootNode from the EditorState
  const root = $getRoot();

  // Get the selection from the EditorState
  const selection = $getSelection();

  // Create a new ParagraphNode
  const paragraphNode = $createParagraphNode();

  // Create a new TextNode
  const textNode = $createTextNode('Hello world');

  // Append the text node to the paragraph
  paragraphNode.append(textNode);

  // Finally, append the paragraph to the root
  root.append(paragraphNode);
});
```


4. **DOM 协调器（DOM Reconciler）**：DOM 协调器负责将编辑器状态的变更高效地反映到 DOM 上的组件。它使用高效的算法来最小化 DOM 操作，提高性能。该机制通常内置于 Lexical 中，无需开发者直接交互。






5. **监听器（Listener）** ：用于订阅编辑器事件，如状态变更、用户操作等。通过监听器，可以执行自定义逻辑，如响应文档的更改。
```js
editor.registerUpdateListener(({ editorState }) => {
    console.log("Editor state updated!", editorState);
});
```

6. **节点转换（Node Transformation）** ：是指对文档树中的节点进行操作的方法，这包括添加、删除、修改节点等。节点操作是通过操作函数来实现的，这些函数定义了如何转换节点。
```js
// When a TextNode changes (marked as dirty) make it bold
editor.registerNodeTransform(TextNode, textNode => {
  // Important: Check current format state
  if (!textNode.hasFormat('bold')) {
    textNode.toggleFormat('bold');
  }
}
```

7. **命令（Command）** ：预定义的操作，它们封装了编辑器操作的逻辑，可以被调用来修改编辑器的状态。命令使得操作逻辑更清晰，易于管理和重用。
```js
import { $insertTextAtSelection } from 'lexical'; // 在当前选择区域插入文本

editor.update(() => {
    $insertTextAtSelection("Inserted text");
});
```

```js
import { createCommand } from 'lexical';

// 定义一个自定义命令
export const CREATE_ALIAS_NODE = createCommand('CREATE_ALIAS_NODE');

export default function AliasPlugin() {
    const [editor] = useLexicalComposerContext();

    useEffect(() => {
        editor.registerCommand(
                CREATE_ALIAS_NODE,
                (options) => {
                    console.log('触发到CREATE_ALIAS_NODE命令')
                    return true;
                },
                COMMAND_PRIORITY_EDITOR,
            )
    }, [editor]);
}
```

8. **选择器（Selection）** ：Lexical 的选择器是 `EditorState` 的一部分，这意味着对于编辑器的每次更新或更改，选择始终与 `EditorState` 的节点树保持一致。
	- `RangeSelection` ：最常见的选择类型，是浏览器 DOM 选择和范围 API 的规范化，包括三个主要属性：
		- `anchor` ：代表一个 `RangeSelection` 点。
		- `focus` ：代表一个 `RangeSelection` 点。
		- `format` ：
	- `NodeSelection` ：表示选择多个任意节点。
	- `TableSelection` (implemented in `@lexical/table`) ：表示像表格一样的网格状选择。它存储选择发生的父节点的键以及起点和终点。
	- `null` ：表示编辑器没有任何活动选择，当编辑器模糊或选择已移动到页面上的另一个编辑器时，这种情况很常见。当尝试在编辑器空间内选择不可编辑的组件时，也会发生这种情况。


### Lexical Nodes

Node 是 Lexical 中的核心概念，其不仅作为 EditorState 的一部分构成可视化编辑器视图，而且还代表编辑器中随时存储的内容的底层数据模型。


#### Base Nodes

Lexical 有一个核心节点，称为 `LexicalNode` ，该节点在内部进行了扩展，以创建 Lexical 的五个基本节点：
- `RootNode`
- `LineBreakNode`
- **`ElementNode`**
- **`TextNode`**
- **`DecoratorNode`**

其中后面三个节点是暴露出来以供个性化扩展的。

##### `RootNode`

EditorState 中只有一个 `RootNode`，它始终位于顶部，代表 `contenteditable` 本身。这意味着 `RootNode` 没有父节点或同级节点。

##### `LineBreakNode`

您的文本节点中永远不应该有 `\n`，而应该使用代表 `\n` 的 `LineBreakNode`，更重要的是，它可以在浏览器和操作系统之间一致地工作。

##### `ElementNode`

用作其他节点的父节点，可以是块级（`ParagraphNode`、`HeadingNode`）和内联（`LinkNode`）的类型，有各种定义其行为的方法，可以在继承时覆盖。

所有的 `div`、`p` 标签都属于 `ElementNode` 节点，所以 `ElementNode` 节点可以包裹其他节点。

##### `TextNode`

文本节点，包含所有的文字的叶子结点，还包括一些特定于文本的属性：
- **`format`** ： `bold`, `italic`, `underline`, `strikethrough`, `code`, `subscript` 和 `superscript` 的任意组合
- **`mode`** ：模式
	- **`token`** ：充当不可变节点，无法更改其内容，并且会一次性删除
	- **`segmented`** ：其内容按段删除（一次一个单词），它是可编辑的，尽管一旦其内容更新，节点就会变为非分段
- **`style`** ：可用于将内联 css 样式应用于文本。


##### `DecoratorNode` 

装饰器结点用于在编辑器内插入任意的视图组件，这个节点是基于 `LexicalNode` 节点开发出来的节点，类型是不可编辑节点，凡是使用 `DecoratorNode` 渲染与框架无关，可以输出来自 React、vanilla js 或其他框架的组件。


#### Node Properties

Lexical Nodes 具有若干属性，这些属性必须是 JSON 可序列化的，永远不应将 function，Symbol，Map，Set 或任何其他与内置对象具有不同原型的属性分配给结点，可以分配给结点的属性类型可以是 `null`，`undefined`，`number`，`string`，`boolean`，`{}` 和 `[]` 。

> [!TIP]
> Lexical 的编程习惯是在属性前加 `__` ，这样可以清楚的表明这是一个私有属性。

如果要添加预期可修改或可访问的属性，则应始终在节点上为该属性创建一组 `get*()` 和 `set*()` 方法。在这些方法中，你需要调用一些非常重要的方法（如 `getWritable()` 和 `getLatest()` ），以确保与 Lexical 内部不可变系统的一致性。

```js
import type {NodeKey} from 'lexical';

class MyCustomNode extends SomeOtherNode {
  __foo: string;

  constructor(foo: string, key?: NodeKey) {
    super(key);
    this.__foo = foo;
  }

  setFoo(foo: string) {
    // getWritable() creates a clone of the node
    // if needed, to ensure we don't try and mutate
    // a stale version of this node.
    const self = this.getWritable();
    self.__foo = foo;
  }

  getFoo(): string {
    // getLatest() ensures we are getting the most
    // up-to-date value from the EditorState.
    const self = this.getLatest();
    return self.__foo;
  }
}
```

所有节点都应具有 `static getType()` 方法和 `static clone()` 方法，Lexical 使用该类型能够在反序列化期间使用其关联的类原型重建节点（对于复制 + 粘贴很重要）。

```js
class MyCustomNode extends SomeOtherNode {
  __foo: string;

  static getType(): string {
    return 'custom-node';
  }

  static clone(node: MyCustomNode): MyCustomNode {
    // If any state needs to be set after construction, it should be
    // done by overriding the `afterCloneFrom` instance method.
    return new MyCustomNode(node.__foo, node.__key);
  }

  constructor(foo: string, key?: NodeKey) {
    super(key);
    this.__foo = foo;
  }

  setFoo(foo: string) {
    // getWritable() creates a clone of the node
    // if needed, to ensure we don't try and mutate
    // a stale version of this node.
    const self = this.getWritable();
    self.__foo = foo;
  }

  getFoo(): string {
    // getLatest() ensures we are getting the most
    // up-to-date value from the EditorState.
    const self = this.getLatest();
    return self.__foo;
  }
}
```


#### Creating custom nodes


##### 继承 `ElementNode`

```js
import {ElementNode, LexicalNode} from 'lexical';

export class CustomParagraph extends ElementNode {
  static getType(): string {
    return 'custom-paragraph';
  }

  static clone(node: CustomParagraph): CustomParagraph {
    return new CustomParagraph(node.__key);
  }

  createDOM(): HTMLElement {
    // Define the DOM element here
    const dom = document.createElement('p');
    return dom;
  }

  updateDOM(prevNode: CustomParagraph, dom: HTMLElement): boolean {
    // Returning false tells Lexical that this node does not need its
    // DOM element replacing with a new copy from createDOM.
    return false;
  }
}

export function $createCustomParagraphNode(): CustomParagraph {
  return new CustomParagraph();
}

export function $isCustomParagraphNode(node: LexicalNode | null | undefined): node is CustomParagraph  {
  return node instanceof CustomParagraph;
}
```


##### 继承 `TextNode`

```js
export class ColoredNode extends TextNode {
  __color: string;

  constructor(text: string, color: string, key?: NodeKey): void {
    super(text, key);
    this.__color = color;
  }

  static getType(): string {
    return 'colored';
  }

  static clone(node: ColoredNode): ColoredNode {
    return new ColoredNode(node.__text, node.__color, node.__key);
  }

  createDOM(config: EditorConfig): HTMLElement {
    const element = super.createDOM(config);
    element.style.color = this.__color;
    return element;
  }

  updateDOM(
    prevNode: ColoredNode,
    dom: HTMLElement,
    config: EditorConfig,
  ): boolean {
    const isUpdated = super.updateDOM(prevNode, dom, config);
    if (prevNode.__color !== this.__color) {
      dom.style.color = this.__color;
    }
    return isUpdated;
  }
}

export function $createColoredNode(text: string, color: string): ColoredNode {
  return new ColoredNode(text, color);
}

export function $isColoredNode(node: LexicalNode | null | undefined): node is ColoredNode {
  return node instanceof ColoredNode;
}
```

##### 继承 `DecoratorNode`

```js
export class VideoNode extends DecoratorNode<ReactNode> {
  __id: string;

  static getType(): string {
    return 'video';
  }

  static clone(node: VideoNode): VideoNode {
    return new VideoNode(node.__id, node.__key);
  }

  constructor(id: string, key?: NodeKey) {
    super(key);
    this.__id = id;
  }

  createDOM(): HTMLElement {
    return document.createElement('div');
  }

  updateDOM(): false {
    return false;
  }

  decorate(): ReactNode {
    return <VideoPlayer videoID={this.__id} />;
  }
}

export function $createVideoNode(id: string): VideoNode {
  return new VideoNode(id);
}

export function $isVideoNode(
  node: LexicalNode | null | undefined,
): node is VideoNode {
  return node instanceof VideoNode;
}
```


#### Node Overrides

核心库拥有和维护一些最常用的 Lexical Nodes，如 `ParagraphNode`、`HeadingNode`、`QuoteNode`、`List(Item)Node` 等。如果想更改 `ListNode` 的行为，通常会继承该类并覆盖方法。但是，如何告诉 Lexical 在 `ListPlugin` 中使用您的 `ListNode` 子类而不是使用核心 `ListNode`？这就是 Node Overrides 可以提供帮助的地方。

```js
const editorConfig = {
    ...
    nodes=[
        // Don't forget to register your custom node separately!
        CustomParagraphNode,
        {
            replace: ParagraphNode,
            with: (node: ParagraphNode) => {
                return new CustomParagraphNode();
            },
            withKlass: CustomParagraphNode,
        }
    ]
}
```


#### Node Transforms

Node Transforms 是响应 EditorState 变化的最有效的机制，Transforms 会在变化传递到 DOM 之前按照顺序执行，即使有多个 Transform 也将只会产生一次 DOM reconciliation （Lexical 生命周期中最昂贵的操作）。

![](../../../Frontend/imgs/Pasted%20image%2020241127095045.png)


**触发**：
- `step 1` ：首先转换叶子节点
	- 如果转换生成了额外的脏节点，则重复 `step 1` 。这样做的原因是，将叶子节点标记为脏节点也会将其所有父元素标记为脏节点。
- `step 2` ：转换元素节点
	- 如果元素转换生成了额外的脏节点，则重复 `step 1` 。 
	- 如果元素转换仅生成额外的脏元素，则仅重复 `step 2` 。 

节点将在对它、其子节点或兄弟节点进行的任何（或大多数）修改上被标记为脏节点。





### Lexical Plugins

Lexical Plugin 是用于 Lexical 编辑器的扩展，它允许开发者在保持核心包轻量的同时，添加额外的功能和自定义行为。这类插件系统设计的目的主要是为了增加编辑器的可扩展性和可定制性，从而让开发者能够根据具体需求调整编辑器的功能。业务的核心功能以及UI等，都可以使用插件实现，每一个功能都是一个插件。

**特点**
1. **分离核心与扩展功能**：在Lexical中，插件承担所有非核心的编辑功能，比如特定格式的输入支持（如Markdown）、复杂的节点操作，甚至集成第三方服务等。核心库因此可以保持轻量和专注于基础编辑任务。
2. **高度自定义**：开发者可以通过创建插件来实现特定的功能，这些功能可以涵盖从简单的格式调整到复杂的交互式元素。例如，一个插件可能允许用户插入和配置图表，而另一个插件则可能增加拼写检查的功能。
3. **易于集成**：插件被设计为易于在Lexical环境中添加和配置。开发者可以通过简单的API调用将插件集成到编辑器实例中，而无需对核心代码进行大量修改或重写。
4. **社区共享**：由于其模块化自然，插件可以被社区开发，共享和再利用。这样不仅可以减少重复工作，还可以利用社区的力量来增强编辑器的功能并快速适应新的需求。
5. **独立更新和维护**：插件可以独立于Lexical核心库进行更新和维护。这意味着添加的新功能或对现有插件的改进可以快速发布，而无需等待编辑器本身的更新。

例如，实现点击复制插件：
```js
import { useLexicalComposerContext } from '@lexical/react/LexicalComposerContext';
import { useEffect } from 'react';
import { createCommand, COMMAND_PRIORITY_EDITOR, $getRoot } from 'lexical';

// 注册一个命令
export const CLICK_COPY = createCommand('CLICK_COPY');

export default function ClickCopy() {
    const [editor] = useLexicalComposerContext();
    
    useEffect(() => {
        editor.registerCommand(
            CLICK_COPY,
            () => {
                const text = $getRoot().getTextContent();
                console.log('编辑器内容', text);
                return true;
            },
            // 事件注册，优先级
            COMMAND_PRIORITY_EDITOR,
        ),
    });
    
    return (
        <button onClick={() => {
            editor.dispatchCommand(CLICK_COPY)
        }}>
            点击复制
        </button>
    )
}
```


实现点击文本替换插件：
```js
import { useLexicalComposerContext } from '@lexical/react/LexicalComposerContext';
import { useEffect } from 'react';
import { createCommand, COMMAND_PRIORITY_EDITOR, $getTextContent } from 'lexical';
// 将自定义节点引入
import { $createCustomDecoratorNode, $isCustomDecoratorNode } from '../nodes/CustomDecoratorNode';
import { $createCustomElementNode, $isCustomElementNode } from '../nodes/CustomElementNode';
import { $createCustomTextNode, $isCustomTextNode } from '../nodes/CustomTextNode';

// 注册一个命令
export const CLICK_UPDATE = createCommand('CLICK_UPDATE');

export default function ClickCopy() {
    const [editor] = useLexicalComposerContext();
    
    useEffect(() => {
        editor.registerCommand(
            CLICK_UPDATE,
            () => {
                const text = $getTextContent(); // 获取到选中的内容
                /**
                * 上边介绍了，ElementNode 相当于是一个容器，DecoratorNode是一个整体，TextNode包裹文本
                * 那么这个需求也是，两边有icon中间是内容，上边是一个按钮，内容还可以编辑，而且还是一个整体，所以就是ElementNode，包裹其他两个节点就可以实现
                **/
                
                // 先创建一个容器节点
                const Container = $createCustomTextNode();
                // 创建一个btn
                const btn = $createCustomDecoratorNode();
                // 创建一个文本节点
                // 其实简单的没有其他的要求用lexical提供的TextNode节点也是可以的，这里为了演示
                const text = $createCustomTextNode(text);
                
                // 直接将两个节点添加到这个容器中，这样一个功能就完成了
                Container.append(btn, text);
                
                console.log('编辑器内容', text);
                return true;
            },
            // 事件注册，优先级
            COMMAND_PRIORITY_EDITOR,
        ),
    });
    
    return (
        <button onClick={() => {
            editor.dispatchCommand(CLICK_UPDATE)
        }}>点击替换文本</button>
    )
}
```


#### With React

Lexical 提供了 `@lexical/react` 包，组织了一系列开箱即用的抽象使 React 用户方便地实现富文本编辑器，并且使用 `JSX` 轻松组合编辑器本身以及所有插件。

```bash
npm install --save lexical @lexical/react
```

##### 示例

`LexicalComposer` 是编辑器的容器组件，用于初始化编辑器实例并管理其生命周期。它利用 React 的 Context 将编辑器实例及其配置传递给其子组件，使子组件能够通过 Lexical 的 Hooks 与编辑器交互。


```js
import {$getRoot, $getSelection} from 'lexical';
import {useEffect} from 'react';

import {AutoFocusPlugin} from '@lexical/react/LexicalAutoFocusPlugin';
import {LexicalComposer} from '@lexical/react/LexicalComposer';
import {RichTextPlugin} from '@lexical/react/LexicalRichTextPlugin';
import {ContentEditable} from '@lexical/react/LexicalContentEditable';
import {HistoryPlugin} from '@lexical/react/LexicalHistoryPlugin';
import {LexicalErrorBoundary} from '@lexical/react/LexicalErrorBoundary';

function Editor() {
  const initialConfig = {
    namespace: 'MyEditor',
    theme: {
      // Theme styling goes here
      //...
    },
    onError: (error: Error) => {
      console.error('Error:', error);
    },
  };

  return (
    <LexicalComposer initialConfig={initialConfig}>
      <RichTextPlugin
        contentEditable={<ContentEditable />}
        placeholder={<div>Enter some text...</div>}
        ErrorBoundary={LexicalErrorBoundary}
      />
      <HistoryPlugin />
      <AutoFocusPlugin />
    </LexicalComposer>
  );
}
```


##### 插件

![](../../../Frontend/imgs/Pasted%20image%2020241119104658.png)

Lexical 没有为其插件定义任何特定接口，最简单的插件是一个接受 `LexicalEditor` 实例并返回清理函数的函数。通过访问 `LexicalEditor`，插件可以通过 `Commands`，`Transforms`，`Nodes` 或其他 API 扩展编辑器。

 > React-based plugins are using Lexical editor instance from `<LexicalComposer>` context.
 > 基于 React 的插件使用的Lexical editor实例是来自 `<LexicalComposer>` 的 context 中。


```js
import { useEffect, useState } from 'react';

import { useLexicalComposerContext } from '@lexical/react/LexicalComposerContext';
import {
    $getSelection,
    COMMAND_PRIORITY_LOW,
    SELECTION_CHANGE_COMMAND,
    TextNode
} from 'lexical';

export default function CustomPlugin() {
    const [editor] = useLexicalComposerContext();

    useEffect(() => {
        // 注册命令：监听光标变化并更新Node
        const unregisterSelectionChangeCommand = editor.registerCommand(
            SELECTION_CHANGE_COMMAND,
            () => {
                editor.update(() => {
                    const selection = $getSelection();
                });
                return false;
            },
            COMMAND_PRIORITY_LOW
        );

        return () => {
            unregisterSelectionChangeCommand(); // 清理注册命令
        };
    }, [editor, lastLineKey]);

    return null;
}
```


### Lexical Listeners

Listener 是一种机制，允许 Editor 实例在发生某项操作时通知用户，所有监听器都遵循反应模式（reactive pattern），可以在将来发生某事时执行操作。

所有监听器还会返回一个函数，该函数可轻松取消注册监听器。

#### `registerUpdateListener`

**触发**：当 Lexical 向 DOM 提交更新。
```js
const removeUpdateListener = editor.registerUpdateListener(({editorState}) => {
  // The latest EditorState can be found as `editorState`.
  // To read the contents of the EditorState, use the following API:

  editorState.read(() => {
    // Just like editor.update(), .read() expects a closure where you can use
    // the $ prefixed helper functions.
  });
});

// Do not forget to unregister the listener when no longer needed!
removeUpdateListener();
```
 该 Listener 传入一个事件回调，参数包括但不限于：
 - `editorState` ：最新更新的编辑器状态
 - `prevEditorState` ：上一个编辑器状态
 - `tags` ：传递给更新的所有标签的集合

##### waterfall updates

**注意**：该模式可能会触发两次 DOM 更新（或者说 waterfall updates），这些更新可以在一次 DOM 更新中完成。这可能会影响性能，而性能在文本编辑器中很重要。
```js
editor.registerUpdateListener(({editorState}) => {
  // Read the editorState and maybe get some value.
  editorState.read(() => {
    // ...
  });

  // Then schedule another update.
  editor.update(() => {
    // Don't do this
  });
});
```
可以使用 [Node Transforms](#Node%20Transforms) 进行替代。



#### `registerTextContentListener`

**触发**：当 Lexical 提交对 DOM 的更新并且编辑器的文本内容与编辑器的先前状态不同，如果更新之间的文本内容相同，则不会向该 Listener 发送通知。
```js
const removeTextContentListener = editor.registerTextContentListener(
  (textContent) => {
    // The latest text content of the editor!
    console.log(textContent);
  },
);

// Do not forget to unregister the listener when no longer needed!
removeTextContentListener();
```



#### `registerMutationListener`

**触发**：当特定类型的 Lexical 节点发生变异（mutation）时，变异有三种状态：
- `created`
- `destroyed`
- `updated`
MutationListener 非常适合跟踪特定类型节点的生命周期，可用于处理与特定类型节点相关的外部 UI 状态和 UI 功能。

```js
const removeMutationListener = editor.registerMutationListener(
  MyCustomNode,
  (mutatedNodes, { updateTags, dirtyLeaves, prevEditorState }) => {
    // mutatedNodes is a Map where each key is the NodeKey, and the value is the state of mutation.
    for (let [nodeKey, mutation] of mutatedNodes) {
      console.log(nodeKey, mutation)
    }
  },
  {skipInitialization: false}
);

// Do not forget to unregister the listener when no longer needed!
removeMutationListener();
```



#### `registerEditableListener`

**触发**：当 editor 的模式发生变化时。
```js
const removeEditableListener = editor.registerEditableListener(
  (editable) => {
    // The editor's mode is passed in!
    console.log(editable);
  },
);

// Do not forget to unregister the listener when no longer needed!
removeEditableListener();
```

编辑器的模式可以通过 `editor.setEditable(boolean)` 来改变。



#### `registerDecoratorListener`

**触发**：当 editor 的装饰器（decorator）对象发生变化时，主要用于外部 UI 框架。
```js
const removeDecoratorListener = editor.registerDecoratorListener(
  (decorators) => {
    // The editor's decorators object is passed in!
    console.log(decorators);
  },
);

// Do not forget to unregister the listener when no longer needed!
removeDecoratorListener();
```



#### `registerRootListener`

**触发**：当 editor 的 root DOM 元素（也就是 content editable）发生变化时收到通知，主要用于将事件侦听器附加到根元素。RootListener 函数在注册时直接执行，然后在任何后续更新时执行。
```js
const removeRootListener = editor.registerRootListener(
  (rootElement, prevRootElement) => {
   //add listeners to the new root element
   //remove listeners from the old root element
  },
);

// Do not forget to unregister the listener when no longer needed!
removeRootListener();
```


### Lexical Commands

Commands 是 Lexical 的一个非常强大的功能，它允许您注册诸如 KEY_ENTER_COMMAND 或 KEY_TAB_COMMAND 等事件的侦听器，并根据上下文在任何地方以您想要的方式对它们做出反应。

此模式对于构建工具栏或复杂的插件和节点（例如 TablePlugin）非常有用，这些插件和节点需要对Selection 、键盘事件等进行特殊处理。

注册命令时，您可以提供优先级并返回 true 以将其标记为“已处理”，从而阻止其他侦听器接收事件。如果您未明确处理命令，则默认情况下，RichTextPlugin 或 PlainTextPlugin 可能会处理该命令。


有一些内置的 Command。

#### `createCommand(...)`

自定义命令。
```js
const HELLO_WORLD_COMMAND: LexicalCommand<string> = createCommand();
```


#### `editor.dispatchCommand(...)`

从可以访问 editor 的任何地方（例如工具栏按钮、事件监听器或插件）发送命令，但大多数核心命令都是从 `LexicalEvents.ts` 发送的。
```js
editor.dispatchCommand(HELLO_WORLD_COMMAND, 'Hello World!');
```


#### `editor.registerCommand(...)`

从任何可以访问 editor 的地方注册一个命令，但重要的是，当不再需要侦听器时，请记住使用其删除侦听器回调来清理该侦听器。

```js
editor.registerCommand(
  HELLO_WORLD_COMMAND,
  (payload: string) => {
    console.log(payload); // Hello World!
    return false;
  },
  LowPriority,
);
```



### 触发顺序

在 Lexical 编辑器中，`Listener` 和 `Command` 的触发顺序通常是这样的：
1. **Listener**：它们是事件驱动的，可以监听特定的事件（如键盘输入、鼠标点击等），并在这些事件发生时被触发。`Listener` 的主要目的是捕捉编辑器状态变化的事件并执行某些操作。例如，它们可以监听文本的改变或某些用户输入的动作。
2. **Command**：命令是在编辑器中执行的操作，通常由用户交互触发，比如执行格式化命令、插入文本或进行撤销/重做操作。命令会在编辑器状态变化时被执行，影响文档内容。

**触发顺序**：
- **事件顺序**：当发生用户操作（比如按键或鼠标事件）时，首先会触发与之关联的 `Listener`，然后再执行命令。
- **命令顺序**：在 `Listener` 执行后，可能会通过一些操作触发相应的 `Command`。命令执行的顺序通常是由其注册顺序决定的。

例如，在按下某个键时：

1. 事件触发时，`Listener` 被首先执行，处理事件并可以更新编辑器状态或执行一些逻辑。
2. 如果 `Listener` 检测到某些条件，可能会触发一个或多个命令（例如插入文本、格式化文本等）。
3. 最后，命令会对编辑器的状态进行修改，更新显示。

总结来说，**Listener 先执行，Command 后执行**，具体顺序也可能根据事件的类型和注册顺序有所不同。


#### 具体顺序

1. **`KEY_DOWN_COMMAND`**：用户按下键时首先触发，处理输入逻辑。
2. **`TextContentListener`**：键盘事件完成后，文本内容更新时触发，监听文本的变化。
3. **`UpdateListener`**：编辑器状态（如光标、选区等）变化后触发，执行更新后的逻辑。






### 推荐语法

#### `editor.read` & `editor.update`

当想要读取或更新词汇节点树时，必须通过 `editor.update(() => {...})` 进行。若只想执行只读操作，可以通过 `editor.read(() => {...})` 或 `editor.getEditorState().read(() => {...})` 读取编辑器状态。

传递给 `update` 或 `read` 调用的闭包（函数）很重要，并且**必须是同步的**，这是获取活动编辑器状态上下文的唯一方式，并提供对编辑器状态节点树的访问权限。


> It is permitted to do nested updates, or nested reads, but an update should not be nested in a read or vice versa.
> 允许进行嵌套更新或嵌套读取，但更新不应嵌套在读取中，反之亦然。

#### `$function`s 

Lexical 推荐使用以 `$` 为前缀的函数（例如 `$getRoot()` ），来表达在这种函数**只可以在**传递给 `update` 或 `read` 调用的**闭包中调用**。

 > Lexical promote using the convention of using `$` prefixed functions (such as `$getRoot()` ) to convey that these functions must be called in this context. Attempting to use them outside of a read or update will trigger a runtime error.
> Lexical 提倡使用以 `$` 为前缀的函数（例如 `$getRoot()` ）的约定，以表示这些函数必须在此上下文中调用，在 `read` 或者 `update` 之外调用会触发 runtime error。

这种 `$function` 优点类似于 React Hooks 的 `useFunction`，同样只能传递同步函数，同样可以调用其他同类函数。

节点转换和命令监听器是通过隐式的 `editor.update(() => {...})` 上下文调用的。

所有 Lexical Node 都依赖于关联的 Editor State，除了少数例外，您只应在 `read` 或 `update` 调用中调用 Lexical Node 的方法和访问其属性（就像 `$function` 一样）。

Lexical Node 上的方法将首先尝试使用节点的唯一键从活动编辑器状态中找到节点的最新（可能可写）版本。逻辑节点的所有版本都具有相同的键。


## Playground

code：[lexical-playground](https://github.com/facebook/lexical/tree/main/packages/lexical-playground)
page：[Lexical Playground](https://playground.lexical.dev/)


## Markdown 定制

在 Lexical 中，可以通过扩展和定制 `MarkdownShortcutPlugin` 和 `TRANSFORMERS` 来实现更高级的 Markdown 支持。


### Lexical `MarkdownShortcutPlugin`

Lexical 的 Markdown 插件注册的是 UpdateListener。






### 自定义 Markdown 规则

`TRANSFORMERS` 是一个数组，其中的每个对象代表一条 Markdown 规则。你可以通过以下步骤添加、修改或删除规则。

例如，支持 `==highlight==` 的语法用于高亮文本：
```js
import { createTextFormatTransformer } from '@lexical/markdown';
import { TextNode } from 'lexical';

// 创建自定义高亮 Transformer
const HighlightTransformer = createTextFormatTransformer(
  'highlight', // 自定义格式标识符
  /==(.*?)==/g, // 匹配正则
  (match) => ({
    format: 'highlight', // 自定义格式名
    text: match[1], // 提取文本
  })
);
```

然后将自定义 Transformer 添加到 `TRANSFORMERS` 中：
```js
const customTransformers = [...TRANSFORMERS, HighlightTransformer];

<MarkdownShortcutPlugin transformers={customTransformers} />;
```

在主题中定义样式：
```js
const theme = {
  text: {
    highlight: 'highlight-class', // CSS 类名
  },
};
```

并在 CSS 中添加样式：
```css
.highlight-class {
  background-color: yellow;
}
```


### 扩展现有规则

找到 `TRANSFORMERS` 中的标题规则并扩展支持的级别：
```js
import { HEADING } from '@lexical/markdown';

const customHeadingTransformer = {
  ...HEADING,
  regExp: /^(#{1,6}) (.*)$/, // 支持 H1-H6
};

const customTransformers = TRANSFORMERS.map((transformer) =>
  transformer === HEADING ? customHeadingTransformer : transformer
);

<MarkdownShortcutPlugin transformers={customTransformers} />;
```

### 自定义导出规则

```js
import { exportMarkdownString } from '@lexical/markdown';

// 自定义导出逻辑
const customExportMarkdownString = (editorState) =>
  exportMarkdownString(editorState, [
    ...TRANSFORMERS,
    {
      export: (node, exportChildren) => {
        if (node.getFormat().includes('highlight')) {
          return `==${node.getTextContent()}==`;
        }
        return null;
      },
      type: 'text',
    },
  ]);
```


### 自定义导入规则

```js
import { importMarkdownString } from '@lexical/markdown';

// 自定义规则：解析双下划线为粗体
const CustomBoldTransformer = {
  regExp: /__(.*?)__/g,
  replace: (match) => ({
    type: 'text',
    format: ['bold'],
    text: match[1],
  }),
};

const customTransformers = [...TRANSFORMERS, CustomBoldTransformer];

const editorState = importMarkdownString(markdownString, customTransformers);
```


### 自定义复杂内容

如果需要添加嵌套内容（如卡片、代码块等），可以使用 Lexical 的组合节点。
```js
import { LexicalNode, TextNode } from 'lexical';

class CardNode extends LexicalNode {
  constructor(content) {
    super();
    this.content = content;
  }

  static getType() {
    return 'card';
  }

  createDOM() {
    const element = document.createElement('div');
    element.className = 'card';
    element.textContent = this.content;
    return element;
  }

  updateDOM() {
    return false;
  }
}

const CardTransformer = {
  type: 'card',
  regExp: /^:::card\s(.*)$/, // 解析自定义卡片语法
  replace: (match) => {
    const cardNode = new CardNode(match[1]);
    return cardNode;
  },
};

// 注册节点
editor.registerNode(CardNode);

// 使用 Transformer
const customTransformers = [...TRANSFORMERS, CardTransformer];

<MarkdownShortcutPlugin transformers={customTransformers} />;
```

添加样式：
```js
.card {
  border: 1px solid #ccc;
  padding: 10px;
  border-radius: 5px;
  background-color: #f9f9f9;
}
```




## 问题

[editor - How can I tweak the Lexical markdown shortcut plugin to actually retain the markdown syntax (rather than replacing) - Stack Overflow](https://stackoverflow.com/questions/78409138/how-can-i-tweak-the-lexical-markdown-shortcut-plugin-to-actually-retain-the-mark)

