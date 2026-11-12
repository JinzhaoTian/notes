**JavaScript 属性描述符（Property Descriptor）** 是描述一个对象属性“行为特征”的普通对象。它决定这个属性：
- 值是什么，或者通过 `getter` / `setter` 读写；
- 是否可写；
- 是否可枚举（`for...in`、`Object.keys` 等是否能看到）；
- 是否可配置（能否删除、修改描述符、改成访问器属性等）。


属性描述符分两类：

1. **数据描述符**：用于普通值属性，字段包括：
```javascript
{
  value: 1,
  writable: true,
  enumerable: true,
  configurable: true
}
```

- `value`：属性值，默认 `undefined`
- `writable`：是否可修改值，默认 `false`
- `enumerable`：是否可枚举，默认 `false`
- `configurable`：是否可配置，默认 `false`

2. **访问器描述符**：用于 `getter` / `setter` 属性，字段包括：
```javascript
{
  get() {},
  set(v) {},
  enumerable: true,
  configurable: true
}
```

- `get`：读取属性时调用的函数，默认 `undefined`
- `set`：设置属性时调用的函数，默认 `undefined`
- `enumerable`、`configurable` 同上

注意：**数据描述符不能同时包含 `value/writable` 和 `get/set`**。

### 示例

```javascript
const obj = {};
Object.defineProperty(obj, 'name', {
  value: 'Tom',
  writable: false,
  enumerable: false,
  configurable: false
});
console.log(obj.name); // Tom
obj.name = 'Jerry';
console.log(obj.name); // Tom，不可写
```

查看属性描述符：
```javascript
Object.getOwnPropertyDescriptor(obj, 'name');
// {
//   value: 'Tom',
//   writable: false,
//   enumerable: false,
//   configurable: false
// }
```

访问器属性示例：
```javascript
const user = {
  _age: 18
};

Object.defineProperty(user, 'age', {
  get() {
    return this._age;
  },
  set(v) {
    if (v >= 0) this._age = v;
  },
  enumerable: true,
  configurable: true
});

console.log(user.age); // 18
user.age = 20;
console.log(user.age); // 20
```


### 常见注意点

1. 用对象字面量或普通赋值创建的属性，默认通常是：
```javascript
{
    writable: true,
    enumerable: true,
    configurable: true
}
```
2. 用 `Object.defineProperty()` 定义属性时，未指定的描述符字段默认是 `false` 或 `undefined`：
```javascript
Object.defineProperty(obj, 'x', { value: 1 });
// writable: false
// enumerable: false
// configurable: false
```

3. `configurable: false` 后，属性不能删除，也不能随意改描述符。例如不能把 `enumerable` 从 `false`改成 `true`。
4. 相关 API：
    - `Object.defineProperty(obj, key, descriptor)`
    - `Object.defineProperties(obj, descriptors)`
    - `Object.getOwnPropertyDescriptor(obj, key)`
    - `Object.getOwnPropertyDescriptors(obj)`
    - `Object.create(proto, descriptors)`