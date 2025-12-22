
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

