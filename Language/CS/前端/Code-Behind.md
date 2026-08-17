**Code-Behind（代码隐藏）** 是一种由微软在早期 .NET 时代（WinForms 和 ASP.NET Web Forms）确立的编程模式。在 WPF 中，它特指**通过 `.xaml.cs` 文件来直接操作和控制 `.xaml` 界面**的编码方式。

## 运行机制

1. **分部类（Partial Class）机制**：`MainWindow.xaml` 和 `MainWindow.xaml.cs` 被编译为同一个类。XAML 中定义的控件（如 `<TextBox x:Name="UserNameBox"/>`）在编译后，会自动生成一个同名的成员变量，因此你可以在 `.xaml.cs` 中直接通过 `UserNameBox.Text` 来读写它的内容。
2. **事件驱动**：交互完全依赖事件（`Click`、`Loaded`、`TextChanged`）。开发者在 XAML 中声明 `Click="SaveButton_Click"`，然后在 `.xaml.cs` 中实现该方法。

## 代码示例

下面是一个完全采用 Code-Behind 模式的登录界面：

1. **`MainWindow.xaml` （视图层）**
```xml
<Window x:Class="WpfDemo.MainWindow">
    <StackPanel>
        <TextBox x:Name="NameBox" />
        <TextBox x:Name="AgeBox" />
        <Button x:Name="SubmitBtn" Click="SubmitBtn_Click" Content="提交" />
    </StackPanel>
</Window>
```

2. **`MainWindow.xaml.cs`（代码隐藏层）**
```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    // 直接在后台代码中处理点击事件
    private void SubmitBtn_Click(object sender, RoutedEventArgs e)
    {
        // 直接通过控件名称操作 UI，获取数据
        string name = NameBox.Text;
        int age = int.Parse(AgeBox.Text);
        
        // 业务逻辑（校验、保存、弹窗）全部混在这里
        if (age < 0)
        {
            MessageBox.Show("年龄不能为负！");
            return;
        }
        
        SaveToDatabase(name, age); // 假设的保存方法
        MessageBox.Show("保存成功！");
    }

    private void SaveToDatabase(string name, int age) { /* ... */ }
}
```


## 优缺点

| 优点                                                         | 缺点（为什么 MVVM 要取代它）                                                                                                |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **上手极快**：对于刚学 C# 的开发者或极小型工具软件，不需要学习绑定、命令、数据上下文等复杂概念，所见即所得。 | **职责严重混淆**：UI 展示逻辑、业务规则（如年龄不能为负）、数据存储（`SaveToDatabase`）全部堆积在同一个 `.xaml.cs` 文件中，随着需求增加，文件轻易膨胀至数千行（俗称“上帝类”）。       |
| **调试直观**：在事件处理方法中打断点，可以直接看到控件的值，查找错误路径非常直接。                | **无法进行单元测试**：`SubmitBtn_Click` 方法依赖于 `NameBox` 和 `AgeBox` 这两个具体的 UI 控件实例。若要测试提交逻辑，必须启动整个 WPF 窗口，无法编写纯内存中的快速单元测试。 |
| **控件访问灵活**：可以直接控制任何控件的颜色、可见性、焦点等视觉属性。                      | **UI 重构困难**：假如把 `NameBox` 改名为 `UserNameTextBox`，所有后台代码中引用它的地方都会报错，耦合性极强。                                         |
|                                                            | **违反开闭原则**：若产品经理要求在点击提交前弹出二次确认框，你必须修改 `SubmitBtn_Click` 内部代码，容易引入新 Bug。                                          |
