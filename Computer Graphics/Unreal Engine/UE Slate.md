Slate 是 Unreal Engine 内置的、用 C++ 编写的声明式 UI 框架，是整个引擎编辑器界面（如 Unreal Editor 的各个面板、菜单、工具栏）的基石，同时也用于构建游戏内的工具界面、菜单和 HUD，高度定制化，深度集成于引擎自身生态。

## 关键特性

1. **声明式语法**：这是 Slate 最显著的特点，通过嵌套 C++ 类型和函数来声明 UI 的层次结构，而不是一步步命令式地创建和设置控件，代码结构非常直观地反映了 UI 的视觉层级。
```cpp
// 一个非常简单的Slate UI声明示例
SNew(SVerticalBox)             // 创建一个垂直盒子容器
+ SVerticalBox::Slot()         // 添加一个槽位
.AutoHeight()                  // 设置槽位属性：自动高度
[
    SNew(SButton)                                // 在槽位里放一个按钮
    .Text(LOCTEXT("ButtonText", "Click Me!"))    // 设置按钮文本
    .OnClicked(this, &MyClass::OnButtonClicked)  // 绑定点击事件
]
+ SVertialBox::Slot()
.FillHeight(1.0f)               // 另一个槽位：填充剩余高度
[
    SNew(STextBlock)            // 放一个文本块
    .Text(LOCTEXT("WelcomeText", "Hello, Slate!"))
];
```

2. **纯 C++ 实现**：Slate 不依赖任何外部 UI 库，它用自己的渲染逻辑在引擎的视口上直接绘制 UI ，这意味着极高的性能和与引擎渲染管线的完美集成。

3. **Unreal Editor 的基石**：你在使用 Unreal Editor 时看到的一切，从内容浏览器、世界大纲视图到细节面板，几乎都是用 Slate 构建的，证明了其强大和稳定性。

4. **高性能和低内存开销**：由于是为高性能引擎量身定制，并且没有外部依赖，Slate 在设计上非常注重效率和资源控制。


## 使用

使用 Slate 进行开发，主要涉及项目配置、控件创建、布局和样式设置等步骤。

1. **项目配置**：要使用 Slate，首先需要正确设置项目的模块依赖。
	- **添加模块依赖**：打开你的项目根目录下的 `[project name].Build.cs` 文件，添加必需的模块
```cs
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore" });
PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });
```

2. **创建与显示控件**
	- **创建自定义控件类**：通常继承自 `SCompoundWidget`，并使用 `SLATE_BEGIN_ARGS` 和 `SLATE_END_ARGS` 宏来定义控件可接收的参数，在 `Construct` 函数中构建 UI。
	- **在游戏中显示控件**：创建控件后，可以将其添加到游戏视口中。
```cpp
// 示例：创建一个简单的文本控件
class SExampleWidget : public SCompoundWidget {
public:
    SLATE_BEGIN_ARGS(SExampleWidget) {}
    SLATE_END_ARGS()
    void Construct(const FArguments& InArgs) {
        ChildSlot
        [
            SNew(STextBlock)
	            .Text(NSLOCTEXT("UIDemo", "HelloWorld", "Hello World!"))
        ];
    }
};
```

```cpp
// 例如在HUD的BeginPlay函数中
void AYourHUD::BeginPlay() {
    Super::BeginPlay();
    TSharedPtr<SExampleWidget> MyWidget = SNew(SExampleWidget);
    if (GEngine && GEngine->GameViewport) {
        GEngine->GameViewport->AddViewportWidgetContent(MyWidget.ToSharedRef());
    }
}
```

3. **理解声明式语法与布局**：Slate 使用声明式语法来组合 UI，主要通过 `SNew` 和 `SAssignNew` 来创建控件，并通过链式调用设置属性和事件。
	- **基础控件创建**
	- **复合布局**：使用如 `SVerticalBox`, `SHorizontalBox` 等容器布局控件，通过 `+Slot()` 添加子控件插槽
```cpp
// 创建一个居中对齐的文本块
SNew(STextBlock)
	.Text(LOCTEXT("Hello", "Hello Slate!"))
	.Justification(ETextJustify::Center)
```

```cpp
// 创建一个水平布局，包含一个文本块和一个按钮
SNew(SHorizontalBox)
	+ SHorizontalBox::Slot() // 第一个槽位
		.AutoWidth()
		[
		    SNew(STextBlock)
			    .Text(FText::FromString("Test Button"))
		]
	+ SHorizontalBox::Slot() // 第二个槽位
		.VAlign(VAlign_Top)
		[
		    SNew(SButton)
			    .Text(FText::FromString("Press Me"))
		]
```

4. **设置样式与处理事件**
	- **样式设置**：可以统一设置并管理控件的视觉风格，例如边框、按钮状态等。
	- **事件处理**：可以为控件绑定事件处理函数，例如按钮点击事件。
```cpp
// 例如，为按钮设置样式
SNew(SButton)
	.ButtonStyle(&FMyStyle::Get().GetWidgetStyle<FButtonStyle>("MyButtonStyle"))
```

```cpp
SNew(SButton)
	.Text(FText::FromString("Click Me"))
	.OnClicked(this, &AYourHUD::OnButtonClicked) // 绑定点击事件处理函数
```

5. **进阶特性探索**
	- **Slate后缓冲区**：允许UI材质对游戏场景进行取样，实现如模糊、失真等后期处理效果。这对于创建全屏UI特效非常有用。
	- **自定义渲染**：通过重写 `OnPaint` 函数，可以实现完全自定义的控件绘制。但这通常比较复杂，仅在现有控件组合无法满足需求时考虑。
    