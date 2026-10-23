Jest 是一个由 Facebook 开发的 JavaScript 测试框架，特别适合 React 应用程序的测试，但也广泛应用于其他 JavaScript 项目。

Jest 提供开箱即用的测试解决方案，大多数情况下无需额外配置即可开始编写测试。

### 使用

1. **断言库**：内置丰富的匹配器(matchers)：
```javascript
expect(value).toBe(4);
expect(value).toEqual({a: 1});
expect(array).toContain('item');
expect(func).toThrow('Error message');
```

2. **Mock 功能**
	- 自动模拟
	- 函数模拟
	- 定时器模拟
	- 模块模拟

3. **快照测试**：特别适合 React 组件测试
```javascript
it('renders correctly', () => {
  const tree = renderer.create(<App />).toJSON();
  expect(tree).toMatchSnapshot();
});
```

### 安装

```bash
npm install --save-dev jest
```

```bash
yarn add --dev jest
```

### 测试示例

1. 测试用例
```javascript
// sum.js
function sum(a, b) {
  return a + b;
}
module.exports = sum;

// sum.test.js
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

2. 运行测试
```bash
npx jest

# 或添加到 package.json
{
  "scripts": {
    "test": "jest"
  }
}
```
### 高级功能

1. **测试覆盖率**：生成代码覆盖率报告
```bash
jest --coverage
```

2. **异步测试**：支持 Promise 和 async/await
```javascript
test('fetches data', async () => {
  const data = await fetchData();
  expect(data).toBe('expected data');
});
```

3. **钩子函数**
```javascript
beforeEach(() => {
  // 每个测试前执行
});

afterAll(() => {
  // 所有测试后执行
});
```

4. **参数化测试**
```javascript
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  expect(a + b).toBe(expected);
});
```
