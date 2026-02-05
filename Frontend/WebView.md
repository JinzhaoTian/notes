WebView（网络视图）是一种在应用程序内部嵌入网页浏览器的组件，允许应用在不跳转到外部浏览器的情况下显示网页内容。

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