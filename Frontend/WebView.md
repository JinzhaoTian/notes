WebView（网络视图）是一种在应用程序内部嵌入网页浏览器的系统组件，允许应用在不跳转到外部浏览器的情况下显示网页内容，和完整的浏览器 App 不同，它本身不包含地址栏、导航按钮等界面元素，只是一个纯粹的页面展示“容器”。


> [!caution] 为什么要引入 WebView 技术？
> 引入 WebView 主要是为了解决移动应用开发中的效率、灵活性和跨平台问题，它在如今的“混合开发”（Hybrid App）模式中扮演核心角色。
> - **提升开发与更新效率**：对需要频繁更新的内容（如活动页面、用户协议），使用 WebView 加载网页，内容更新无需发布新版本App。同时，它允许前端开发者利用熟悉的 Web 技术栈为App开发功能，降低了对原生开发者的依赖。
> - **增强应用的动态性与灵活性**：WebView 可以轻松展示图文、视频等富媒体信息。通过 Web 页面与原生代码的交互，它既能拥有Web的灵活性，又能调用硬件功能，兼具原生能力。
> - **实现跨平台与代码复用**：通过 WebView，开发者可以“一次编写，随处运行”，将同一套 Web 代码用于不同平台（Android、iOS等），显著节省开发成本。
> - **降低开发门槛与成本**：对于展示型页面，使用 WebView 的开发成本远低于原生开发。此外，WebView 作为系统组件，开发者无需额外集成即可使用。
> - **技术演进与性能提升**：现代 WebView（如Android的System WebView）已基于Chromium项目，与Chrome共用渲染引擎，支持硬件加速，并有独立的更新渠道，可及时获得性能和安全修复。


## 核心概念

1. **嵌入式浏览器**：本质上是一个简化版的浏览器内核，集成在 App 中。
2. **混合开发**：常用于混合应用（Hybrid App），结合了原生功能和网页技术（HTML/CSS/JavaScript）。


## 主要特点

1. **无需离开应用**：用户可直接在应用内浏览网页，体验更流畅。
2. **可定制与交互**
    - 开发者可以控制 WebView 的界面（如隐藏地址栏）。
    - 支持 JavaScript 与原生代码（Java/Kotlin、Swift/Objective-C）交互，实现双向通信。
3. **跨平台内容**：适合加载动态更新的内容（如新闻、活动页面），无需频繁更新整个 App。


## 实现示例

- **Android**：使用系统组件 [`android.webkit.WebView`](https://developer.android.com/reference/android/webkit/WebView)。
- **iOS**：使用 [`WKWebView`](https://developer.apple.com/documentation/webkit/wkwebview)（推荐）或旧版的 `UIWebView`。
- **跨平台框架**：Flutter、React Native 等也提供 WebView 组件。