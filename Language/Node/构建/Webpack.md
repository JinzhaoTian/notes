Webpack 是一个现代化的 **JavaScript 模块打包工具（Module Bundler）**，主要用于将多个分散的 JavaScript 文件（及其依赖的其他资源，如 CSS、图片等）打包成一个或多个优化后的文件，以便在浏览器中高效加载。

### 核心功能

1. **模块化打包**
    - 支持 `ES Modules`（`import/export`）、`CommonJS`（`require/module.exports`）等模块系统。
    - 自动分析依赖关系，构建依赖图（Dependency Graph），并打包成最终文件。

2. **代码拆分（Code Splitting）**
    - 支持按需加载（懒加载），优化首屏加载速度。

3. **加载器（Loaders）**
    - 处理非 JavaScript 文件（如 `.css`、`.scss`、`.png`、`.ts` 等），将其转换为 Webpack 可识别的模块。
    - 示例：`babel-loader`（转译 ES6+）、`css-loader`（处理 CSS）、`file-loader`（处理文件）。

4. **插件（Plugins）**
    - 提供更强大的功能，如代码压缩、环境变量注入、HTML 模板生成等。
    - 示例：`HtmlWebpackPlugin`（生成 HTML）、`MiniCssExtractPlugin`（提取 CSS）。

5. **开发优化**
    - 开发服务器（`webpack-dev-server`）支持热更新（HMR）。
    - 生产模式自动优化（Tree Shaking、代码压缩等）。


### 基本配置

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js', // 入口文件
  output: {
    filename: 'bundle.js', // 输出文件名
    path: path.resolve(__dirname, 'dist'), // 输出目录
  },
  module: {
    rules: [
      {
        test: /\.js$/, // 匹配 .js 文件
        exclude: /node_modules/,
        use: 'babel-loader', // 使用 Babel 转译
      },
      {
        test: /\.css$/, // 匹配 .css 文件
        use: ['style-loader', 'css-loader'], // 处理 CSS
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html', // 生成 HTML 文件
    }),
  ],
  devServer: {
    static: './dist', // 开发服务器目录
    hot: true, // 热更新
  },
  mode: 'development', // 开发模式（生产模式用 'production'）
};
```


### 应用场景

1. **单页应用（SPA）**：如 React、Vue、Angular 项目。
2. **多页应用（MPA）**：通过配置多个入口文件实现。
3. **静态资源优化**：压缩图片、提取 CSS、Tree Shaking 等。
4. **微前端架构**：配合 Module Federation 实现模块共享。


