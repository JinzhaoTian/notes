来源：[Application Management Overview - WPF .NET Framework | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/desktop/wpf/app-development/application-management-overview?view=netframeworkdesktop-4.8)

Windows Presentation Foundation（WPF）是一个演示框架，可用于开发以下类型的应用程序：
- **独立应用程序**（传统风格的 Windows 应用程序，构建为可执行程序集，安装到客户端计算机并从客户端计算机运行）。
- XAML 浏览器应用程序（XBAP）（由导航页面组成的应用程序，构建为可执行程序集并由 Web 浏览器（如 Microsoft Internet Explorer 或 Mozilla Firefox）托管）。
- 自定义控件库（包含可重用控件的不可执行程序集）。
- 类库（包含可重用类的不可执行程序集）。

为了构建这组应用程序，WPF 实现了许多服务。

## Application Management

可执行的 WPF 应用程序通常需要一组核心功能，包括以下内容：
- 创建和管理通用应用程序基础结构（包括创建入口点方法和 Windows 消息循环以接收系统和输入消息）。
- 跟踪应用程序的生命周期并与之交互。
- 检索和处理命令行参数。
- 共享应用程序范围属性和 UI 资源。
- 检测和处理未处理的异常。
- 返回退出代码。
- 管理独立应用程序中的窗口。
- 跟踪 XAML 浏览器应用程序（XBAP）中的导航以及具有导航窗口和框架的独立应用程序。
- 这些功能由 Application 类实现，您可以使用应用程序定义将其添加到应用程序中。

### Implementing an Application Definition

典型的 WPF 应用程序的定义是使用 XAML 和 code-behind 实现的。这允许您使用 XAML 以声明方式设置应用程序属性、资源和注册事件，同时在 code-behind 中处理事件并实现特定于应用程序的行为。

以下示例显示了如何使用标记和代码隐藏实现应用程序定义：
```xml
<Application 
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
  x:Class="SDKSample.App" />
```

```C#
using System.Windows;

namespace SDKSample
{
    public partial class App : Application { }
}
```

要使 XAML 标记文件和 code-behind 文件协同工作，需要进行以下操作：
- 在 XAML 中，Application 元素必须包含 `x:Class` 属性。构建应用程序时，XAML 标记文件中的 `x:Class` 会让 MSBuild 创建一个从 Application 派生且具有 `x:Class` 属性指定的名称的 `partial` 类。
  XML 命名空间声明（`xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"`）需要添加 到 XAML 架构。

- 在 code-behind 中，该类必须是具有与标记中的 `x:Class` 属性指定的名称相同的名称的部分类，并且必须从 Application 派生。

### Application Lifetime

WPF 应用程序的生命周期，是由 `Application` 类引发的若干事件进行标记，以明白应用程序何时启动、何时激活、何时停用以及何时关闭。

![](_imgs/Pasted%20image%2020250107153034.png)

#### Starting an Application

调用 `Run` 并初始化应用程序后，应用程序即可运行，此时 `Startup` 事件触发。

##### 方法一
```xml
<Application
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  x:Class="SDKSample.App" 
  Startup="App_Startup" />
```

```C#
using System.Windows;

namespace SDKSample
{
    public partial class App : Application
    {
        void App_Startup(object sender, StartupEventArgs e)
        {
            // Application is running
            MainWindow = new MainWindow();
			MainWindow.Show();
        }
    }
}
```

大多数独立 Windows 应用程序在开始运行时都会打开一个窗口。


##### 方法二

每个 WPF 应用程序都由一个 `Main` 方法开始，通常是由 `Application` 类生成的。如果在代码中没有显式定义 `Main` 方法，那通常会由编译器自动生成，如：
```C#
[STAThread]
public static void Main()
{
    var app = new App();
    app.Run();
}
```

在 `Run` 方法启动后，`OnStartup` 方法会被调用。这个方法是 `Application` 类的一个虚方法，允许在应用程序启动时执行自定义的初始化操作，
```C#
protected override void OnStartup(StartupEventArgs e)
{
    base.OnStartup(e);

	MainWindow = new MainWindow();
	MainWindow.Show();
}
```


##### 方法三

使用 `StartupUri`，

```xml
<Application 
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  x:Class="SDKSample.App"
  StartupUri="MainWindow.xaml" />
```


#### Application Activation and Deactivation

Windows 允许用户在应用程序之间切换，如使用 `ALT+TAB` 组合键。只有当应用程序具有用户可以选择的可见窗口时，才可以切换到该应用程序。当前选定的窗口是活动窗口（也称为前台窗口），是接收用户输入的窗口。具有活动窗口的应用程序是活动应用程序（或前台应用程序）。在以下情况下，应用程序将成为活动应用程序：
- 它已启动并显示窗口。
- 用户通过选择应用程序中的窗口从另一个应用程序切换。

可以通过处理 `Application.Activated` 事件来检测应用程序何时变为活动状态。同样，在以下情况下，应用程序可能会变为非活动状态：
- 用户从当前应用程序切换到另一个应用程序。
- 当应用程序关闭时。

可以通过处理 `Application.Deactivated` 事件来检测应用程序何时变为非活动状态。


#### Application Shutdown

应用程序的生命周期在关闭时结束，关闭可能由于以下原因发生：
- 用户关闭每个窗口。
- 用户关闭主窗口。
- 用户通过注销或关闭来结束 Windows 会话。
- 已满足特定于应用程序的条件。

为了帮助您管理应用程序关闭，`Application` 提供了 `Shutdown` 方法、`ShutdownMode` 属性以及 `SessionEnding` 和 `Exit` 事件。


## Windows

用户通过窗口与 WPF 独立应用程序交互。窗口的目的是承载应用程序内容并公开通常允许用户与内容交互的应用程序功能。在 WPF 中，窗口由 Window 类封装，该类支持：
- 创建和显示窗口。
- 建立所有者/被拥有窗口关系。
- 配置窗口外观（例如，大小、位置、图标、标题栏文本、边框）。
- 跟踪窗口的生命周期并与之交互。

Window 支持创建一种特殊类型的窗口（称为对话框）的功能。可以创建模式和非模式类型的对话框。

为了方便、可重用性和跨应用程序的一致用户体验，WPF 公开了三个常见的 Windows 对话框：OpenFileDialog、SaveFileDialog 和 PrintDialog。

消息框是一种特殊类型的对话框，用于向用户显示重要的文本信息，以及询问简单的“是/否/确定/取消”问题。您可以使用 MessageBox 类来创建和显示消息框。

###  Implementing a Window

典型窗口的实现包括外观和行为，其中外观定义窗口在用户眼中的外观，行为定义窗口在用户与其交互时的工作方式。在 WPF 中，您可以使用代码或 XAML 标记实现窗口的外观和行为。

但是，一般来说，窗口的外观是使用 XAML 标记实现的，其行为是使用代码隐藏实现的，如下所示，
```xml
<Window 
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    x:Class="SDKSample.MarkupAndCodeBehindWindow">
  
  <!-- Client area (for content) -->
  
</Window>
```

```c#
using System.Windows;

namespace SDKSample
{
    public partial class MarkupAndCodeBehindWindow : Window
    {
        public MarkupAndCodeBehindWindow()
        {
            InitializeComponent();
        }
    }
}
```

为使 XAML 标记文件（XAML markup file）和代码隐藏文件（code-behind file）协同工作，需要满足以下要求：
- 在 XAML 中，Window 元素必须包含 `x:Class` 属性。构建应用程序时，标记文件中的 `x:Class` 会导致 MSBuild 创建一个从 Window 派生且具有 `x:Class` 属性指定的名称的 `partial` 类。
  XML 命名空间声明（`xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"` ）需要添加到 XAML 中。生成的 `partial` 类实现 `InitializeComponent` 方法，该方法被调用来**注册事件**并设置在 XAML 中实现的属性。

- 在 code-behind 中，窗口类必须是具有与 XAML 中的 `x:Class` 属性指定的名称相同的名称的 `partial` 类，并且必须从 Window 派生。这样，code-behind 文件就可以与构建应用程序时为 XAML 文件生成的 `partial` 类相关联。

- 在 code-behind 中，Window 类必须实现一个调用 `InitializeComponent` 方法的构造函数。`InitializeComponent` 由 XAML 文件生成的 `partial` 类实现，用于注册事件并设置标记中定义的属性。


完成此配置后，可以专注于在 XAML 中定义窗口的外观并在 code-behind 中实现其行为。

### Window Lifetime

与任何类一样，窗口的生命周期从首次实例化时开始，之后被打开、激活和停用，最终关闭。

![](_imgs/Pasted%20image%2020250107153226.png)

#### Opening a Window

