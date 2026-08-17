MVVM（Model-View-ViewModel）是一种软件架构设计模式，其核心目标是**将用户界面（View）的开发与业务逻辑（Model）的开发分离开来**。


1. **Model（模型层）**：负责管理应用程序的数据和业务逻辑，其独立于用户界面，例如从数据库或网络 API 获取的数据。
2. **View（视图层）**：即用户界面，负责数据的展示。在 MVVM 中，View 是被动的，它不包含业务逻辑，只负责将 ViewModel 的数据呈现给用户。
3. **ViewModel（视图模型层）**：这是 MVVM 的核心，作为 View 和 Model 之间的桥梁。它从 Model 获取数据并处理，转换成 View 可以直接使用的形式。同时，它还负责处理用户的交互行为。

MVVM 最大的特点是**双向数据绑定**，这意味着 ViewModel 中的数据变化会自动更新到 View 上；反之，用户在 View 上的操作（如输入）也会自动同步到 ViewModel 中。开发者无需再手动操作 DOM 来更新界面，极大地提高了开发效率。


## 对比

| 特性       | MVC (Model-View-Controller)                                                             | MVP (Model-View-Presenter)                                        | MVVM (Model-View-ViewModel)                                                                            |
| -------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **核心思想** | 引入 Controller 来协调 Model 和 View 。                                                        | 用 Presenter 完全替代 Controller ，并彻底隔离View 和 Model 。                  | 用ViewModel替代Presenter，并通过数据绑定实现自动同步。                                                                   |
| **通信方式** | **View与Controller交互**，Controller 更新 Model， Model 再更新 View 。View 和 Model 可能直接交互。         | **View 与 Presenter 双向通信**，Presenter 再与Model 交互。View 和 Model 完全隔离。 | **View 与 ViewModel 通过数据绑定自动同步**。ViewModel 与 Model 交互，View 和 Model 完全隔离。                                |
| **优点**   | 概念简单，适合小型应用。                                                                            | 解决了 MVC 中 View 和 Model 耦合的问题，提高了可测试性。                             | **解耦最彻底**，View 和 Model 完全隔离。**自动化程度最高**，数据绑定免去了大量手动更新 UI 的代码。**可测试性最强**， ViewModel 不依赖 UI ，可进行纯粹的单元测试。 |
| **缺点**   | **职责不清**，Controller（尤其在 Android 中常由 Activity 承担）容易变得臃肿。**测试困难**，因为业务逻辑与 Android 框架紧密耦合。 | **代码量增加**，Presenter 需要手动处理所有 View 和Model 的交互，容易变得庞大。              | **学习曲线较陡**，需要理解数据绑定、命令等概念。**调试稍复杂**，数据流向不如 MVC 直观。                                                     |
| **适用场景** | 逻辑简单的页面或小型项目。                                                                           | 需要更清晰架构但不想引入数据绑定复杂性的中型项目。                                         | 复杂的、交互频繁的用户界面，尤其适合需要高效开发和良好可维护性的中大型项目。                                                                 |


## 开发实践

要有效运用 MVVM，可以参考以下实践：

1. **保持职责单一清晰**
	- **View只负责显示**：View中只应包含 UI 布局和样式，避免编写任何业务逻辑。
	- **ViewModel处理交互与状态**：ViewModel 应包含所有与界面相关的状态和用户交互逻辑。
	- **Model专注数据和业务**：Model 应只关心数据的结构、获取和存储，不依赖 UI 或ViewModel。

2. **拥抱数据绑定和命令**
	- **充分利用数据绑定**：这是 MVVM 的精髓。应使用框架提供的声明式绑定，将 ViewModel 的属性直接绑定到 UI 元素上。
	- **使用命令（Command）处理事件**：对于按钮点击等UI事件，应使用 `Command` 或类似机制将其绑定到 ViewModel 的方法上，而不是在 View 的后台代码中编写事件处理器。

3. **提升可测试性**
	- **为 ViewModel 编写单元测试**：由于 ViewModel 不依赖于具体的 UI（如 Activity 或ViewController），你可以轻松地为其编写纯 JVM 或纯逻辑的单元测试，而无需启动应用或模拟器。
	- **依赖注入（DI）**：在 ViewModel 中，通过构造函数等方式注入 Model 或 Repository 的实例，而不是直接`new`出来。这能让单元测试时更方便地注入模拟（Mock）对象。

4. **管理好生命周期和异步操作**
	- **利用框架的生命周期感知组件**：在Android中，使用`ViewModel`和`LiveData`可以很好地处理屏幕旋转等配置变化导致的数据丢失问题。在iOS中，可以利用 `Combine` 或 `RxSwift` 等框架。
	- **保持界面响应性**：执行网络请求或数据库读写等耗时操作时，务必使用异步方式（如协程、RxJava、Async/Await），并在操作完成后通过数据绑定更新UI，避免阻塞主线程。

5. **组织代码结构**
	- **创建Repository层**：对于复杂应用，建议在 Model 层引入 Repository 模式，作为数据访问的唯一入口。ViewModel 只与 Repository 交互，而不直接进行网络或数据库操作。
	- **模块化**：将不同的功能划分为独立的模块，每个模块内部采用 MVVM 架构，这有助于大型项目的并行开发和维护。


### Model 与 ViewModel 转换

Model（数据实体）和 ViewModel（UI状态）的对应关系极少是一一映射的，通常存在**结构重组**（扁平化嵌套对象）、**类型转换**（时间戳转日期）和**数据组合**（多表联查）的复杂需求。

针对这部分的设计，业界的主流共识是：**引入独立的 Mapper（映射器）层，将转换逻辑从 ViewModel 中抽离出来，并严格区分“展示映射”和“提交映射”两个方向**。

#### 核心原则

**引入 Mapper 层，而非在 ViewModel 中硬写**。不要直接在 ViewModel 里写 `user.name + user.age` 的拼接逻辑，也不要在 ViewModel 的构造函数里做大量转换。

- **错误做法**：ViewModel 直接持有 Model，并在获取数据后手动 `set` 给 LiveData/StateFlow。
- **正确做法**：ViewModel 调用 Repository 获取 Model，然后将 Model 丢给 Mapper，Mapper 返回一个全新的 **UI 模型（UIModel / ViewState）**，ViewModel 只需将这个 UIModel 赋值给 State。

```kotlin
// 伪代码示意
class UserViewModel(
    private val repo: UserRepository,
    private val mapper: UserMapper // 注入映射器
) : ViewModel() {
    fun loadUser() {
        val model = repo.getUser() // 数据库/网络 Model
        val uiState = mapper.mapToUi(model) // 核心转换交给 Mapper
        _state.value = uiState
    }
}
```

#### 具体形式

根据项目大小和语言特性，推荐以下三种实现方式：

1. **方案一：专用映射器类（最推荐，适合中大型项目）**  
	- 创建独立的 `XxxMapper` 类，通过构造函数注入工具类（如日期格式化工具）。这样做**职责单一**，且极其便于编写单元测试。
2. **方案二：扩展函数（适合 Kotlin / Swift 项目）**  
	- 为 Model 类编写扩展函数，如 `fun UserEntity.toUIModel(): UserUIModel`。这种写法非常简洁，但不适合内部需要依赖复杂工具类的场景（不好注入依赖），适合简单的字段一对一拷贝。
3. **方案三：领域模型（Domain Model）过渡（适合复杂业务）** 
	- 引入 Domain 层作为中间层。Repository 返回 Domain Model（纯净的业务对象），Mapper 再将 Domain Model 转为 UI Model。这么做的好处是，当 UI 展示逻辑频繁变动时，Domain Model 保持稳定，只需修改 Mapper 即可，不会污染业务层。

#### 双向映射

很多开发者只关注“数据展示”（Model → ViewModel），却忘了“数据提交”（ViewModel → Model）。两者设计必须分开：

|映射方向|业务场景|设计要点|
|---|---|---|
|**正向映射（Model → UI）**|从数据库/网络加载数据并显示在屏幕上。|**做“加法”**：将时间戳转为“3分钟前”，将 `firstName+lastName` 拼接为全名，将枚举转为中文描述。|
|**反向映射（UI → Model）**|用户编辑表单后点击提交/保存。|**做“减法”**：将 UI 中的格式化字符串（如“2026-08-17”）解析回时间戳，将下拉框选中的中文转回后端需要的枚举值。|

**特别注意**：反向映射往往涉及**合法性校验**。建议将校验逻辑放在 ViewModel 的业务流中，而不是 Mapper 里。Mapper 只负责“干净的数据结构转换”，不负责“判断数据是否合法”。


#### 警惕

1. **不要在 Mapper 中执行 IO 操作**：Mapper 只做内存中的格式化和结构转换，绝对**不能**在里面查数据库、读 SharedPreferences 或调用网络 API。如果需要根据 ID 查名字，请在 Repository 层组合好数据再传给 Mapper。
2. **不要将业务逻辑塞入 Mapper**：例如“如果金额大于100显示红色，否则显示绿色”。这种**展示逻辑（View Logic）** 可以放在 Mapper 里；但“如果金额大于100，则不能提交订单”这种**业务规则（Business Rule）** 必须放在 ViewModel 或 Domain 层。
3. **避免过度映射**：如果 Model 和 ViewModel 字段完全一致（纯列表展示），可以跳过 Mapper 层，直接复用 Model 作为 UI 状态，不必为了“架构而架构”。



## 示例

以下是一个在 C# 中使用 WPF 框架的完整 MVVM 示例，演示了从数据展示、用户交互到双向绑定的完整流程。

### 1. 创建 Model（数据模型）

`Model` 只负责存储数据，不包含任何业务逻辑。这里创建一个 `Person` 类。

```csharp
// Models/Person.cs
namespace WpfMvvmDemo.Models
{
    public class Person
    {
        public string Name { get; set; }
        public int Age { get; set; }
    }
}
```

### 2. 创建 ViewModel（视图模型）

`ViewModel` 是 MVVM 的核心，它需要做三件事：
1. **暴露数据**：将 `Model` 的数据包装成可供 View 绑定的属性。
2. **实现通知**：通过 `INotifyPropertyChanged` 接口，在属性值变化时通知 View 更新。
3. **处理命令**：通过实现 `ICommand` 接口，处理按钮点击等用户交互。

为了简洁并演示常见的 `RelayCommand` 模式，这里手动实现了一个 `RelayCommand` 类。

```csharp
// ViewModels/RelayCommand.cs
using System;
using System.Windows.Input;

namespace WpfMvvmDemo.ViewModels
{
    // 一个通用的命令实现，用于将 View 的事件绑定到 ViewModel 的方法
    public class RelayCommand : ICommand
    {
        private readonly Action _execute;
        private readonly Func<bool> _canExecute;

        public RelayCommand(Action execute, Func<bool> canExecute = null)
        {
            _execute = execute ?? throw new ArgumentNullException(nameof(execute));
            _canExecute = canExecute;
        }

        public event EventHandler CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }

        public bool CanExecute(object parameter) => _canExecute == null || _canExecute();
        public void Execute(object parameter) => _execute();
    }
}
```

```csharp
// ViewModels/MainViewModel.cs
using System.ComponentModel;
using System.Windows.Input;
using WpfMvvmDemo.Models;

namespace WpfMvvmDemo.ViewModels
{
    public class MainViewModel : INotifyPropertyChanged
    {
        private Person _person;

        public MainViewModel()
        {
            // 初始化数据
            _person = new Person { Name = "张三", Age = 25 };
            
            // 初始化命令
            IncreaseAgeCommand = new RelayCommand(IncreaseAge);
        }

        // 暴露给 View 绑定的属性
        public string Name
        {
            get => _person.Name;
            set
            {
                if (_person.Name != value)
                {
                    _person.Name = value;
                    OnPropertyChanged(nameof(Name));
                }
            }
        }

        public int Age
        {
            get => _person.Age;
            set
            {
                if (_person.Age != value)
                {
                    _person.Age = value;
                    OnPropertyChanged(nameof(Age));
                }
            }
        }

        // 命令属性，用于按钮绑定
        public ICommand IncreaseAgeCommand { get; }

        // 命令的执行方法：年龄+1
        private void IncreaseAge()
        {
            Age++;
        }

        // INotifyPropertyChanged 实现
        public event PropertyChangedEventHandler PropertyChanged;
        protected void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```

### 3. 创建 View（视图）

`View` 是用户界面，通过 XAML 定义，并使用 `{Binding}` 声明式地将控件与 `ViewModel` 的属性或命令绑定起来。

关键在于将 `Window` 的 `DataContext` 设置为 `ViewModel` 的实例，这样所有绑定才能生效。

```xml
<!-- MainWindow.xaml -->
<Window x:Class="WpfMvvmDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:WpfMvvmDemo.ViewModels"
        Title="MVVM 人员信息管理" Height="180" Width="300">
    
    <!-- 将窗口的数据上下文设置为 ViewModel -->
    <Window.DataContext>
        <local:MainViewModel />
    </Window.DataContext>

    <Grid Margin="20">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="60"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- 姓名 -->
        <TextBlock Text="姓名：" Grid.Row="0" Grid.Column="0" VerticalAlignment="Center"/>
        <!-- 双向绑定，用户输入会实时更新 ViewModel -->
        <TextBox Text="{Binding Name, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" 
                 Grid.Row="0" Grid.Column="1" Margin="5"/>

        <!-- 年龄 -->
        <TextBlock Text="年龄：" Grid.Row="1" Grid.Column="0" VerticalAlignment="Center"/>
        <!-- 单向绑定，只显示 ViewModel 中的数据 -->
        <TextBlock Text="{Binding Age}" 
                   Grid.Row="1" Grid.Column="1" Margin="5" VerticalAlignment="Center"/>

        <!-- 按钮，通过 Command 绑定到 ViewModel 的方法 -->
        <Button Content="增加年龄" 
                Command="{Binding IncreaseAgeCommand}"
                Grid.Row="2" Grid.ColumnSpan="2" 
                HorizontalAlignment="Center" Margin="5" Padding="10,5"/>
    </Grid>
</Window>
```

> [!caution] 
> 在标准的 WPF 项目中，**`MainWindow.xaml.cs` 文件是默认存在的**（在 Visual Studio 中它作为 `MainWindow.xaml` 的“子节点”折叠在下方）。
> 
> 在严格的 MVVM 模式中，`MainWindow.xaml` 中没有业务逻辑，这个 `MainWindow.xaml.cs`（即 [Code-Behind](Code-Behind.md)，代码隐藏文件）**应该保持极其干净，甚至完全为空**。

`MainWindow.xaml.cs` 文件的内容默认如下，且**不需要修改**：

```csharp
using System.Windows;

namespace WpfMvvmDemo
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent(); // 仅此一行，加载 XAML 界面
        }
    }
}
```


> [!tip] 为什么 MVVM 要求 `*.xaml.cs`  “空白”？
> 在 MVVM 中，`.xaml.cs` 被视为 **View（视图层）的“外部容器”**。我们要求它保持空白，是因为：
> - **职责分离**：`.xaml` 负责“长什么样”（静态布局），`.xaml.cs` 负责“做什么的交互逻辑”。为了能够对业务逻辑进行**纯单元测试**（不启动 UI 窗口），所有交互逻辑和数据处理必须放在 **ViewModel** 中，而不是放在 `.xaml.cs` 的事件处理器（如 `Button_Click`）里。
> - **避免“胖视图”**：如果你把点击按钮的逻辑写在 `.xaml.cs` 中，这个文件就变成了传统的 Code-Behind 模式，直接破坏了 MVVM 的双向绑定和命令机制。
> 
> **那么什么时候可以在 `.xaml.cs` 里写代码？**
> 
> 虽然原则上是“保持空白”，但在实际桌面开发中，有两类 UI 专属逻辑不得不放在这里。请记住：**凡是涉及 UI 控件本身特有的视觉行为（且无法通过 Binding 解决）的，才放这里**；凡是涉及数据的，绝对不放这里。
> 
> **合法场景**（可以写）：
> - **窗口生命周期操作**：窗口加载完成后自动获取键盘焦点（`Focus()`）。
> - **纯视觉动画控制**：触发 Storyboard（情节提要）动画，或关闭子窗口（`DialogResult`）。
> - **第三方 UI 控件适配**：某些第三方图表或地图控件没有依赖属性，必须通过代码订阅事件来初始化。
> - **获取物理屏幕尺寸**：计算窗口弹出位置等。
>  
>  ```csharp
>  public partial class MainWindow : Window
>  {
>     public MainWindow()
>     {
>         InitializeComponent();
>         
>         // 1. 视图特有的行为：窗口居中，不影响 ViewModel
>         this.WindowStartupLocation = WindowStartupLocation.CenterScreen;
>         
>         // 2. 加载完成后设置焦点（注意：这里调用的是 View 中的控件名，不涉及业务数据）
>         this.Loaded += (s, e) => MyTextBox.Focus();
>     }
>  }
>  ```

