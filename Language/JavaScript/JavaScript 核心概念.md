
## 文档对象模型（DOM）

文档对象模型（DOM）是针对HTML和XML文档的一个 **API** 。DOM描绘了一个层次化的节点树，允许开发人员添加、移除和修改页面的某一部分。

```JavaScript
document.getElementById('id属性值');             // 返回拥有指定id的第一个对象的引用
document.getElementsByClassName('class属性值');  // 返回拥有指定class的对象集合
document.getElementsByTagName('标签名');         // 返回拥有指定标签名的对象集合
document.getElementsByName('name属性值');        // 返回拥有指定名称的对象结合
document.querySelector('CSS选择器');             // 仅返回第一个匹配的元素

object.getAttribute(attribute)                  // 获取属性
object.setAttribute(attribute,value)            // 设置属性

document.write("<p>p标签</p>")                  // 创建元素
document.createElement("标签");                 // 创建元素节点
```

## 语法标准

CommonJs 是一种模块化的规范，包括现在的 Node.js 里面也采用了部分 CommonJs 语法在里面，后来 Es6 版本正式加入了 Es Module。ES 模块是 JavaScript 的标准，而 CommonJS 是 Node.js 中的默认模块。

现在 Node.js 支持两种语法。

### CommonJS

```JavaScript
// index.js
module.exports.name = "jinzhao"
module.exports.age = 24


// other.js
let data = require("./index.js")


// other.js
let lists = ["./index.js", "./config.js"]
lists.forEach((url) => require(url)) // 动态导入
```

- `require()` 是同步加载模块。
- CommonJs 可以动态加载语句，代码发生在运行时;
- CommonJs 混合导出，还是一种语法，只不过不用声明前面对象而已，当我导出引用对象时之前的导出就被覆盖了
- CommonJs 导出值是拷贝，可以修改导出的值，这在代码出错时，不好排查引起变量污染.

### Es module

```JavaScript
// index.js
export let num = 0;
export function add() {
    ++ num
}

// other.js
import { num, add } from "./index.js"
```

- `import` 是异步加载模块，静态编译时加载，有独立的模块依赖解析。
- Es Module是静态的，不可以动态加载语句，只能声明在该文件的最顶部，代码发生在编译时;
- Es Module混合导出，单个导出，默认导出，完全互不影响;
- Es Module导出是引用值之前都存在映射关系，并且值都是可读的，不能修改。

