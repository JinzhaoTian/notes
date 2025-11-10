
## 变量和类型

JavaScript是弱变量类型的语言，变量只需要用var来声明，但是仍然内置了变量的类型。

1. 变量类型：变量均为对象。
    1. 数据类型：string，number，boolean，object，function，symbol
    2. 对象类型：Object，Date，Array
    3. 不包含任何值的数据类型：null，undefined
2. 创建变量：
在 ES6 之前，JavaScript 中永远都是用var来定义变量。

```JavaScript
var 变量名 = new 类型名；

var 数组名 = new Array();
var 数组名 = new Array(值1，值2，...);
var 数组名 = [值1，值2，...];

var 对象名 = new Object();
var 对象名 = {
  属性1 : 值1,
  属性2 : 值2,...
}

let 变量名 = new 类型名；               // 定义的限定范围内作用域的变量
```

3. 比较：
    1. `==`：等于
    2. `===`：绝对等于（值和类型均相等）
    3. `!=`：不等于
    4. `!==`：不绝对等于



