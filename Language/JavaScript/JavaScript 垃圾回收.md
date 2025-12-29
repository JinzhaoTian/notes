JavaScript 使用自动垃圾回收，主要通过以下算法：
1. **标记清除（Mark-and-Sweep）**
```javascript
// 示例：对象变得不可访问时被回收
let obj = { name: "test" }; // 创建对象
obj = null; // 原对象现在不可达，将被垃圾回收

// 循环引用也可以被处理
function createCycle() {
    let a = {};
    let b = {};
    a.b = b;
    b.a = a;  // 循环引用
    return "done";
}
createCycle(); // 函数执行后，a和b都不可达，会被回收
```

2. **引用计数（Reference Counting）**：已基本淘汰
```javascript
// 引用计数的问题：循环引用无法释放
function problem() {
    let objA = {};
    let objB = {};
    objA.ref = objB;  // objB 引用计数: 2
    objB.ref = objA;  // objA 引用计数: 2
    // 即使函数结束，引用计数也不为0，导致内存泄漏
}
```


