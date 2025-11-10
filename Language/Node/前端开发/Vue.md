Vue 是一个用于构建用户界面的渐进式 JavaScript 框架，被设计为可以自底向上逐层应用，适用于构建单页应用（SPA）和复杂的动态网页。

**核心特点**

1. **渐进式框架**
    - Vue 可以逐步集成到项目中，既可以用作轻量级的视图层库，也可以扩展为完整的单页应用框架（配合 Vue Router、Vuex/Pinia 等）。
2. **响应式数据绑定**
    - 基于 **数据劫持（Object.defineProperty）或 Proxy（Vue 3）** 实现自动更新视图，数据变化时 UI 同步更新。
3. **组件化开发**
    - 将页面拆分为可复用的组件，每个组件包含自己的 HTML、CSS 和 JavaScript，便于维护和协作。
4. **虚拟 DOM**
    - 通过高效的 Diff 算法减少直接操作真实 DOM 的开销，提升性能。
5. **指令系统**
    - 提供 `v-if`、`v-for`、`v-bind`、`v-on` 等模板指令，简化 DOM 操作。
6. **单文件组件（SFC）**
    - 使用 `.vue` 文件将模板、逻辑和样式封装在一起，例如：
        <template>
          <div>{{ message }}</div>
        </template>
```vue
<template>
	<div>{{ message }}</div>
</template>
<script>
	export default {
		data() { return { message: "Hello Vue!" }; }
	};
</script>
<style scoped>
	div { color: red; }
</style>
```
7. **生态系统**
    - 官方支持库（如 Vue Router、Vuex/Pinia、Vite）和丰富的社区插件。

