[Next.js](https://nextjs.org/) 是一个基于 React 的流行框架，用于构建服务器端渲染（SSR）和静态网站生成（SSG）的现代 Web 应用程序。它由 Vercel 团队开发，提供了许多开箱即用的功能，使开发者能够更轻松地创建高性能、SEO 友好的 Web 应用。

## 主要特点

1. **[服务端渲染](服务端渲染.md)**：可以在服务器端生成页面内容，在首次加载时向客户端返回预渲染的 HTML，提升 SEO 和初始加载性能。
2. **静态生成 (SSG)**：可以在构建时生成静态页面，并在运行时快速提供这些页面，适用于博客等内容相对静态的网站。
3. **API 路由**：内置 API 路由功能，允许你在同一项目中定义后端 API 路由，简化前后端集成。
4. **自动代码拆分**：Next.js 会自动将代码分割，只加载用户访问页面所需的代码，优化性能。
5. **图片优化**：内置的 `next/image` 组件可以优化图片的加载和显示，包括自动延迟加载、压缩和格式转换。
6. **文件路由**：Next.js 使用文件系统作为路由机制，每个页面都对应 `pages/` 文件夹中的一个文件。
7. **TypeScript 支持**：开箱即用地支持 TypeScript，让开发者能轻松编写类型安全的代码。
8. **热重载 (Fast Refresh)**：在开发时提供即时预览和模块热替换，改进开发者体验。

Next.js 适用于需要良好 SEO、性能优化和用户体验的网站与应用，尤其是内容丰富的站点、博客、电子商务平台等。



## 原理

1. **Next.js 使用文件系统作为路由机制**，每个 `pages` 目录中的文件都会自动成为一个路由，支持嵌套路由和动态路由。这种机制简化了路由配置，使开发者能够快速创建和管理页面。
2. **Next.js 在请求时生成页面**，服务器接收到请求后，会运行相应的页面组件，并在服务器上生成 HTML。生成的 HTML 会直接返回给客户端，这提高了首次加载速度和 SEO 优化。
3. **Next.js 允许开发者在 `pages/api` 目录下创建 API 路由**，这些路由会处理请求并返回响应。API 路由实际上是 Node.js 服务器上的函数，可以用于处理数据操作。


### 路由机制

Next.js 的路由机制是**基于文件系统**的，主要特点包括：

1. **页面文件**：在 `pages` 或 `app` 目录下的每个文件都对应一个路由。例如，`pages/index.js` 对应 `/` 路由，`pages/about.js` 对应 `/about` 路由。
2. **动态路由**：使用方括号定义动态路由。比如，`pages/posts/[id].js` 可以匹配 `/posts/1`, `/posts/2` 等等，`id` 是动态参数，可以在组件中通过 `useRouter` 获取。
3. **嵌套路由**：可以通过在子目录中创建文件来实现嵌套路由。例如，`pages/blog/index.js` 对应 `/blog`，而 `pages/blog/[slug].js` 对应 `/blog/:slug`。
4. **API 路由**：在 `pages/api` 目录下的文件用于创建 API 路由。每个文件对应一个 API 路由，可以处理 GET、POST 等请求。
6. **路由钩子**：可以使用 `next/router` 模块中的钩子，例如 `useRouter`，来获取当前路由的信息、编程式导航等。
7. **中间件**：在 Next.js 13 及以上版本中，可以使用中间件（`middleware.js`）在请求处理之前执行一些操作，比如身份验证或重定向。


 **文件名的特殊含义**：
- `layout.tsx`：通常位于特定目录下，Next.js 会在渲染**该目录下的页面**时，自动将其包装在这个布局组件中。
- `page.tsx`：在 `pages` 或 `app` 目录下的文件，如果命名为 `page.jsx` 或 `page.tsx`，其将会被视为路由页面，**缺少这个文件将不会被视作路由**。
- `error.tsx`：通常放在页面目录中，可以根据需求自定义错误处理逻辑和展示样式。
- `loading.tsx`：放置在页面或布局目录中，Next.js 在相关页面加载时自动显示这个组件。
- `middleware.js`：放在根目录或特定路由目录下，可以用于控制请求的流向。


### 渲染

#### 静态生成

静态生成（Static Generation，SSG）会在构建时（build time）生成 HTML 文件，并在每次请求时复用。它适用于内容不会频繁变化的页面，比如博客文章、产品列表等。

静态生成的两种方式是：
- **getStaticProps**：在构建时预先获取数据，用于生成静态页面。每次请求返回的 HTML 都相同，可以实现快速的页面加载。
```js
// pages/post/[id].js
export async function getStaticProps(context) {
  const { id } = context.params;
  const post = await fetchPostData(id); // 获取数据
  return {
    props: {
      post,
    },
  };
}
```
- **getStaticPaths**：用于动态路由，通过指定可能的路径来生成动态页面的静态内容。适合为每一个可能的动态 URL 生成静态页面。
```js
export async function getStaticPaths() {
  const paths = await fetchAllPostIds(); // 获取所有路径
  return {
    paths,
    fallback: false, // 如果路径不存在则显示404
  };
}
```

生成的静态内容在构建后可以部署到 CDN 上，能够极大地提高加载速度。

#### 服务端渲染

服务端渲染（Server-Side Rendering，SSR）会在每次请求时生成页面 HTML 内容。Next.js 提供了 **getServerSideProps** 函数，在请求时从服务器获取数据并渲染页面。SSR 适合需要实时数据的页面，比如用户个性化内容、实时更新的数据页面等。

- **getServerSideProps**：在每次请求时运行，用于在服务器端获取数据并生成 HTML，确保数据是最新的。
- 缺点是相较静态生成有一定的响应时间，因为每次请求都需要在服务器上生成内容。

```js
// pages/profile.js
export async function getServerSideProps(context) {
  const data = await fetchDataForProfile(); // 请求数据
  return {
    props: {
      data,
    },
  };
}
```


#### 客户端渲染

客户端渲染（Client-Side Rendering，CSR）是在页面加载后，通过 JavaScript 在客户端获取和渲染内容。适用于非关键性数据的加载，例如在页面加载后显示的用户评论、分页数据等。可以通过 React 的钩子（如 `useEffect`）实现数据的获取。

- **SWR** 和 **React Query** 等库可以进一步简化客户端数据获取与缓存。

#### 增量静态生成

增量静态生成（Incremental Static Regeneration，ISR）允许在静态生成的基础上对页面进行更新，而无需重新构建整个应用。Next.js 通过 **revalidate** 参数指定页面的重新生成间隔，使静态页面在后台自动更新。

- ISR 适用于数据更新较为频繁但不需要实时更新的页面，比如新闻站点首页等。页面会在后台自动再生成并缓存。

#### 动态渲染机制：混合渲染

混合渲染（Hybrid Rendering），Next.js 支持对不同页面选择不同的渲染方式，因此在一个应用中可以混合使用静态生成、服务端渲染和客户端渲染。通过这种方式，Next.js 可以针对不同页面需求优化渲染性能和用户体验。

### "use client"

在 Next.js 中，`"use client"` 是一种指令，用于指定一个文件（通常是组件文件）在客户端渲染。自 Next.js 13 推出 **App Router** 以来，默认情况下，组件都在服务器端渲染（Server Components），而 `"use client"` 的引入允许开发者在需要时指定客户端渲染（Client Components）。

#### 使用场景

当组件包含以下特性时，应该使用 `"use client"` 指令：

- **使用 React 的客户端钩子**，例如 `useState`、`useEffect`、`useRef` 等。因为这些钩子只能在客户端环境下运行。
- **与浏览器相关的操作**，例如 `document` 或 `window` 对象的访问。服务器端并没有这些对象，所以需要在客户端渲染。
- **依赖浏览器 API** 的逻辑，比如 `localStorage`、`sessionStorage`，这些只存在于客户端环境中。



## 使用

```bash
npx create-next-app@latest
```


### 特定文件目录

1. `components\` ：通常包含项目的所有 UI 组件。这里的组件通常会分为基础组件（可复用的小组件）和页面级组件。
2. `contexts\` ：用于存放 React 上下文（Context API）相关代码，集中管理应用程序的全局状态，避免 props 层层传递。
	- **常见用途**：比如用户认证状态、主题设置、语言偏好等，可以通过 `contexts` 来提供和管理。
3. `hooks\` ：通常包含自定义的 React hooks，用于处理状态管理、事件处理或其他常见逻辑。这些 hooks 可以在多个组件中复用，从而避免重复代码。
	- **常见用途**：
		- 管理组件的内部状态（例如：sidebar 折叠状态、模态框的打开和关闭）。
		- 处理复杂的用户交互逻辑（例如：拖拽和放置、鼠标和键盘事件）。
		- 提供 API 数据的管理（例如：使用 `useFetch` 来获取和缓存数据）。
		- 增加功能性，如表单验证、延迟加载等。
4. `lib\` ：通常包含与项目相关的实用工具函数、配置文件和库代码，这些代码不直接依赖于组件的逻辑，但为项目提供了基础功能支持。
	- **常见用途**：
		- 存放通用的实用函数（例如：格式化日期、深拷贝对象）。
		- 项目级别的配置（例如：API 基础路径、颜色配置）。
		- 外部库的封装，简化组件对外部依赖的调用（例如：对 `axios`、`date-fns` 等库的封装）。
		- 导出项目常用的类型定义、接口等。
5. `types\` ：通常用于存放 TypeScript 类型定义文件。这是 TypeScript 项目中特别重要的一个文件夹，用于集中管理和复用类型。
	- **常见用途**：项目中自定义的接口、类型定义、枚举等。
6. `services\` ：用于封装与服务器的交互逻辑，通常包括 API 请求函数或与外部服务的集成代码。


### 业务逻辑拆分

在前端项目中，`onClick` 或 `handle` 函数的存放位置取决于函数的作用范围、复用需求和项目结构。以下是一些常见的组织方式和最佳实践：

1. **直接放在组件内部**

如果点击事件的处理逻辑只在一个组件中使用，并且逻辑较简单，可以将 `onClick` 或 `handle` 函数直接放在该组件内部。

- **适用场景**：处理逻辑简单，且只对单个组件有影响的事件。
```ts
import React from 'react';

function MyButton() {
    const handleClick = () => {
        console.log("Button clicked!");
    };

    return <button onClick={handleClick}>Click Me</button>;
}

export default MyButton;
```

2. **父组件中定义事件处理函数并通过 props 传递**

如果需要从父组件管理状态或逻辑，可以将 `handle` 函数放在父组件中，然后通过 `props` 将该函数传递给子组件。

- **适用场景**：事件处理逻辑需要访问父组件的状态或方法，或者影响多个子组件。
```ts
import React, { useState } from 'react';
import MyButton from './MyButton';

function ParentComponent() {
    const [count, setCount] = useState(0);

    const handleButtonClick = () => {
        setCount(count + 1);
    };

    return (
        <div>
            <p>Button clicked {count} times</p>
            <MyButton onClick={handleButtonClick} />
        </div>
    );
}

export default ParentComponent;
```

3. **自定义 Hooks 中定义事件逻辑**

对于复杂的逻辑或复用性较强的事件处理函数，可以将它们放在自定义 Hooks 中，这样可以在多个组件中复用相同的逻辑。

- **适用场景**：需要在多个组件中复用的逻辑，或涉及较复杂的状态管理。
```ts
// hooks/useButtonLogic.ts
import { useState } from 'react';

export function useButtonLogic() {
    const [count, setCount] = useState(0);

    const handleClick = () => {
        setCount((prev) => prev + 1);
    };

    return { count, handleClick };
}

// MyButton.tsx
import React from 'react';
import { useButtonLogic } from './hooks/useButtonLogic';

function MyButton() {
    const { count, handleClick } = useButtonLogic();

    return (
        <div>
            <button onClick={handleClick}>Click Me</button>
            <p>Clicked {count} times</p>
        </div>
    );
}

export default MyButton;
```


4. **在 `contexts` 中集中管理业务逻辑**

如果 `onClick` 或 `handle` 函数需要在全局范围使用（如控制应用中的用户认证、主题设置等），可以使用 `contexts` 文件夹中的上下文（Context）管理。

- **适用场景**：需要在整个应用程序或多个组件中共享和管理的逻辑，或依赖全局状态。
```ts
// contexts/ButtonContext.tsx
import React, { createContext, useContext, useState } from 'react';

const ButtonContext = createContext(null);

export function ButtonProvider({ children }) {
    const [count, setCount] = useState(0);
    const handleClick = () => setCount((prev) => prev + 1);

    return (
        <ButtonContext.Provider value={{ count, handleClick }}>
            {children}
        </ButtonContext.Provider>
    );
}

export const useButton = () => useContext(ButtonContext);

// MyButton.tsx
import React from 'react';
import { useButton } from './contexts/ButtonContext';

function MyButton() {
    const { count, handleClick } = useButton();

    return (
        <div>
            <button onClick={handleClick}>Click Me</button>
            <p>Clicked {count} times</p>
        </div>
    );
}

export default MyButton;
```


5. **`services` 文件夹中定义复杂的业务逻辑**

在大型应用中，如果 `onClick` 的处理逻辑涉及到 API 请求、数据处理、格式化等复杂业务逻辑，可以将这些操作封装在 `services` 中，再由 `onClick` 函数调用这些服务。

- **适用场景**：需要访问后端、处理复杂业务规则，或者逻辑过于复杂而不适合直接写在组件或 hooks 中。
```ts
// services/userService.ts
export async function fetchUserData(userId: string) {
    const response = await fetch(`/api/users/${userId}`);
    const data = await response.json();
    return data;
}

// MyComponent.tsx
import React, { useState } from 'react';
import { fetchUserData } from './services/userService';

function UserProfile({ userId }) {
    const [userData, setUserData] = useState(null);

    const handleFetchUserData = async () => {
        const data = await fetchUserData(userId);
        setUserData(data);
    };

    return (
        <div>
            <button onClick={handleFetchUserData}>Fetch User Data</button>
            {userData && <p>{JSON.stringify(userData)}</p>}
        </div>
    );
}

export default UserProfile;
```


**总结** ：
- **组件内部**：简单的 `onClick` 逻辑，作用范围仅限于单个组件。
- **父组件传递**：需要管理子组件或多个组件的状态。
- **自定义 Hooks**：复杂或需要复用的逻辑。
- **Contexts**：全局状态或逻辑，多个组件共享。
- **Services**：复杂的业务逻辑，涉及到数据处理或 API 调用。

根据业务逻辑的复杂性和复用需求，可以灵活地选择合适的存放位置。这样可以让代码更易于维护、复用和理解。