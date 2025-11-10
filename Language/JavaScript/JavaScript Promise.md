
Javascript语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。
- 同步模式：后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的。
- 异步模式：每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。


### Callback

在 JavaScript 中，回调是一个函数，可以作为参数传递给另一个函数，在某个特定的事件发生或者异步操作完成时被调用。

回调函数经常用于处理异步操作，例如 AJAX 请求、定时器或事件处理程序。通过将回调函数作为参数传递给异步函数，我们可以在异步操作完成时执行特定的逻辑。

> AJAX（Asynchronous JavaScript and XML，异步的JavaScript 和 XML），向服务器异步发送和接收数据，然后用JavaScript解析。

尽管回调函数是一种常见的异步编程模式，但它也有一些缺点，例如**回调地狱**（ [Callback Hell](http://callbackhell.com/) ）和难以处理错误。因此，现代 JavaScript 中也出现了其他异步编程技术，如 Promise、async/await 等，以更好地管理异步代码。

JavaScript 中的**回调机制**与 Node.js 的**事件循环机制密切相关**。Node.js 是基于事件驱动的，它使用了事件循环来处理异步操作和事件。回调函数是在 Node.js 中处理异步操作时的一种常见模式，而事件循环则是负责调度和执行这些回调函数的机制之一。

在 Node.js 中，事件循环是一个持续运行的循环，它不断地检查事件队列中是否有待处理的事件，如果有，则会将这些事件推入调用栈中执行相应的回调函数。

当您在 Node.js 中执行异步操作时，比如读取文件、发送网络请求等，通常会提供一个回调函数，在异步操作完成时调用该回调函数。Node.js 将这些回调函数放入事件队列中，当事件循环检测到队列中有回调函数时，它将逐个取出并执行这些回调函数，从而完成异步操作的处理。






### Promise


Promise 是 JavaScript 中用于处理异步操作的对象，它代表了一个异步操作的最终完成（或失败）及其结果的值。使用 Promise 可以更清晰地组织和管理异步代码，避免了回调地狱（callback hell）问题，使异步操作的代码更具可读性和可维护性。

一个 Promise 可以处于以下三种状态之一：
1. **Pending（进行中）：** 初始状态，表示异步操作还在进行中，尚未完成。
2. **Fulfilled（已完成）：** 表示异步操作已成功完成。
3. **Rejected（已拒绝）：** 表示异步操作失败或出错。

一个 Promise 可以通过调用 `resolve()` 或 `reject()` 方法来转换为 Fulfilled 或 Rejected 状态。一旦 Promise 进入其中一个终态，它就会固定在那个状态，不可再改变。

```js
// 创建一个 Promise
const myPromise = new Promise((resolve, reject) => {
    // 模拟异步操作
    setTimeout(() => {
        const randomNumber = Math.random();
        if (randomNumber > 0.5) {
            resolve(randomNumber); // 将 Promise 置为 Fulfilled 状态
        } else {
            reject(new Error('Random number is less than 0.5')); // 将 Promise 置为 Rejected 状态
        }
    }, 1000);
});

// 使用 Promise
myPromise.then((result) => {
    console.log('Promise resolved with result:', result);
}).catch((error) => {
    console.error('Promise rejected with error:', error);
});
```

使用 `then()` 方法来注册 Fulfilled 状态的处理函数，使用 `catch()` 方法来注册 Rejected 状态的处理函数。当 Promise 的状态发生改变时，对应的处理函数就会被调用。

Promise 还支持链式调用，可以通过 `then()` 方法的返回值再次返回一个 Promise 对象，从而实现链式调用的方式处理异步操作。

此外，Promise 还提供了一些静态方法：
1. `.then()`：调用 then 可以为实例注册两种状态的回调函数，当实例的状态为 fulfilled，会触发第一个函数执行,当实例的状态为 rejected，则触发第二个函数执行。
2. `.all()`： all 方法提供了并行执行异步操作的能力，在 all 中所有异步操作结束后才执行回调。
3. `.race()`：等到第一个Promise改变状态就开始执行回调函数。
4. `.finally()`：finally 方法只有当状态变化的时候才会执行，可以用来做一些程序的收尾工作，比如操作文件的时候关闭文件流。


### await/async

  
在 JavaScript 中，可以使用 `await` 关键字与 `async` 函数结合使用来处理 Promise。`async` 函数用于定义一个异步函数，它返回一个 Promise 对象，而 `await` 关键字用于等待 Promise 对象的解决（即状态变为 Fulfilled）并获取其结果。

如果异步操作成功完成，将返回结果；如果异步操作失败，则**捕获错误并抛出**。




### Promisify

将回调包装成 Promise 是一种常见的做法，特别是在处理旧有的异步 API 或者库时。

> 参考： [Redis 的 Promise 包装](../../Database/Redis/使用.md#Promise%20包装) 。

