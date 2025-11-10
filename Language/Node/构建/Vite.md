[Vite](https://cn.vitejs.dev/) 是一个现代的前端构建工具，旨在提供更快的开发体验。它使用原生 ES 模块加载，支持热更新和快速构建，适合用于 Vue、React 等框架的项目。

Vite 将 `index.html` 视为源码和模块图的一部分。Vite 解析 `<script type="module" src="...">` ，这个标签指向你的 JavaScript 源码。甚至内联引入 JavaScript 的 `<script type="module">` 和引用 CSS 的 `<link href>` 也能利用 Vite 特有的功能被解析。另外，`index.html` 中的 URL 将被自动转换，因此不再需要 `%PUBLIC_URL%` 占位符了。
