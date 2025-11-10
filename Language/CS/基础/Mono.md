Mono 是一个开源的跨平台开发框架，主要用于运行和开发基于 .NET 的应用程序，由 Xamarin 开发，旨在让开发者可以在非 Windows 系统（如 Linux、macOS 等）上运行 .NET 应用。Mono 现在是 .NET Foundation 的一部分。

![](_imgs/Pasted%20image%2020240912100802.png)

主要特性：
1. **跨平台支持**：Mono 允许开发者编写一次代码，然后可以在多个平台上运行，包括 Windows、Linux、macOS、Android、iOS 等。
2. **兼容 .NET Framework**：Mono 与微软的 .NET Framework 高度兼容，因此可以运行大部分 .NET 应用。
3. **支持多种编程语言**：Mono 支持 C# 等基于 .NET 的编程语言。
4. **用于移动开发**：Mono 是 Xamarin 的基础技术，通过它，开发者可以使用 C# 编写跨平台的 Android 和 iOS 应用。
5. **性能优化**：Mono 包含 Just-In-Time (JIT) 和 Ahead-Of-Time (AOT) 编译器，能够优化应用程序的性能，尤其是在移动和嵌入式设备上。


# Xamarin

Xamarin 是一个开源的跨平台应用开发框架，允许开发者使用 C# 和 .NET 来构建适用于 Android、iOS 和 Windows 的本机应用程序。它是由 Xamarin 公司开发的，后来被微软收购并集成到 .NET 生态系统中。Xamarin 的核心目标是通过共享代码库实现跨平台开发，同时保留原生应用的性能和用户体验。

Xamarin 允许开发者使用单一代码库（主要是 C#）来构建跨 Android、iOS 和 Windows 平台的应用程序，开发者可以使用相同的业务逻辑代码（如后端服务调用、数据库访问、业务逻辑等），只需要编写少量与平台相关的代码来处理 UI 和平台特定的功能。

Xamarin 曾是微软的跨平台移动开发的主要框架，但随着 .NET 6 和 .NET MAUI（Multi-platform App UI）的推出，Xamarin 将逐渐被 .NET MAUI 取代。MAUI 是 Xamarin.Forms 的演进版本，支持更多平台（如 Windows 和 macOS），并简化了跨平台 UI 开发。

Xamarin 支持已于 2024 年 5 月 1 日结束。

# 与 .NET Framework、.NET Core/.NET 5+ 的区别

- **.NET Framework** 
	- 只能运行在 Windows 平台，针对 Windows 平台的闭源开发框架。
	- 提供了一个丰富的类库和公共语言运行时（Common Language Runtime，CLR），主要用于开发 Windows 桌面和服务器端应用程序。
- **Mono** 
	- 可以在非 Windows 系统上运行，特别是 Linux 和 macOS，开源项目。
	- 与 .NET Framework 保持高度兼容。
	- 提供了自己的运行时，称为 Mono Runtime，并实现了自己的版本的 CLR 和类库。
	- 是跨平台移动开发的核心技术。
	- 在某些场景下使用 AOT（Ahead-of-Time） 编译，特别是在移动平台上，如 iOS。iOS 禁止 JIT（Just-In-Time）编译，因此 Mono 提供了 AOT 编译选项。此外，Mono 也支持 JIT 编译（如在 Linux、macOS 和 Windows 上）。
- **.NET Core/.NET 5+** 
	- 支持 Windows、Linux 和 macOS，跨平台开源。
	- 基于 CLR（Common Language Runtime）
	- 更适合用于 Web、云端和跨平台桌面应用开发，但在移动开发上并没有完全取代 Mono。
	- 默认使用 JIT 编译，但也可以通过 ReadyToRun 或 AOT 来预先编译代码，以优化性能和启动时间。


