AST（Abstract Syntax Tree，抽象语法树）是一种树状的数据结构，用来表示源代码的语法结构。树上的每个节点都表示源代码中的一个构造，例如表达式、语句、声明、类型等。

如，对于 C++ 代码：
```cpp
int result = a + b * 5;
```

其 AST 结构大致如下：
```
            VarDecl（变量声明：‘result’）
               |
          BinaryOperator（赋值：‘=’）
           /              \
   DeclRefExpr（变量 ‘a’)   BinaryOperator（加法：‘+’）
                            /              \
                   DeclRefExpr（变量 ‘b’)   BinaryOperator（乘法：‘*’）
                                            /              \
                                   IntegerLiteral（整数：‘5’)  ... 
```


## 主要特点

1. **保留了丰富的语义信息**：它不仅知道那里有一个标识符，还知道这个标识符指的是哪个变量、它的类型是什么、是在哪里声明的。
2. **忠实于源代码**：与某些编译器（如 GCC）的 AST 不同，Clang 的 AST 非常注重保留源代码的原始结构，包括宏展开、括号位置等。这使得它非常适合用于源代码分析和转换工具。
3. **包含所有细节**：即使是隐式生成的代码（如默认构造函数、析构函数、隐式类型转换），也会在 AST 中明确表示出来。

## 使用

> 官方文档：[Clang AST](https://clang.llvm.org/docs/IntroductionToTheClangAST.html)

### 通过命令行工具

1. **使用 `-Xclang -ast-dump` 标志**：
```bash
clang -Xclang -ast-dump -fsyntax-only my_file.cpp
```
- **参数**：
	- `-fsyntax-only`：只进行语法分析，不生成代码。
	- `-Xclang -ast-dump`：将传递给 Clang 内部的 `-ast-dump` 选项，它会将 AST 打印到标准输出。
		- `-ast-dump=json`：将 AST 以 JSON 格式输出，便于其他工具解析。


### 使用 LibTooling（官方推荐）

[LibTooling](https://clang.llvm.org/docs/LibTooling.html) 是 Clang 提供的一个库，可用于创建强大的、独立的代码分析和重构工具，这些工具可以对 C/C++ 代码进行语法和语义分析，提供了操作 AST、源代码位置、诊断信息等所需的几乎所有接口。

#### 核心概念

1. `ASTFrontendAction`：一个接口，用于在编译过程中对每个翻译单元执行自定义操作。
2. `ASTConsumer`：由 `ASTFrontendAction` 创建，用于接收 AST 并对其进行操作。
3. `RecursiveASTVisitor`：一个辅助类，可以递归地遍历 AST 树，并在遇到特定类型的节点（如 `FunctionDecl`, `CXXRecordDecl`, `IfStmt` 等）时调用相应的访问方法（如 `VisitFunctionDecl`, `VisitIfStmt`）。
4. `CommonOptionsParser`：clang 工具中用于处理常见命令行选项的类。
5. `ClangTool`


##### `clang::tooling::ClangTool`

`clang::tooling::ClangTool` 是 LLVM/Clang 基础设施中一个高级别的、方便使用的类，它简化了基于 Clang 的工具（如代码检查器、重构工具、转换器）的创建过程。

**核心工作**是帮你管理复杂的编译命令、文件处理，并允许你专注于对源代码的抽象语法树（AST）进行分析或转换的逻辑。

直接使用 LibTooling 的底层 API 需要开发者处理很多繁琐的细节，比如：
- 解析复杂的编译命令行（`-I`, `-D`, `-std=c++11` 等）。
- 为每个源文件创建和管理 `clang::CompilerInstance` 对象。
- 正确处理虚拟文件系统（VFS）等。

于是 `ClangTool` 应运而生，它封装了这些底层细节，提供了一个干净、简单的接口。


###### 主要职责

1. **编译命令管理**：
    - 它接受一个编译命令数据库（CompilationDatabase），最常见的是从项目根目录的 `compile_commands.json` 文件中自动加载。
    - 这个数据库包含了每个源文件编译时所需的精确 flags（包含路径、宏定义等）。这意味着你的工具可以像项目的构建系统（如 CMake, Bear, Bazel）一样“看待”代码，确保分析结果准确无误，没有解析错误。
2. **批量文件处理**：
    - 你可以指定要处理的一系列源文件（例如，整个项目、某个目录下的文件或特定的文件列表）。
    - `ClangTool` 会自动化地为每个文件设置正确的编译环境，并逐个进行处理。
3. **运行 AST Consumers**：
    - 你不需要直接创建和管理 `CompilerInstance`。
    - 相反，你定义一个 `clang::ast_matchers::MatchFinder::MatchCallback` 对象（用于匹配特定的 AST 模式并执行操作）或一个自定义的 `clang::ASTConsumer` 对象。
    - 然后将这个消费者通过 `clang::tooling::runToolOnCode` 或更常用的 `clang::tooling::newFrontendActionFactory` 包装成一个 `FrontendAction`，最后传递给 `ClangTool::run` 方法。
    - `ClangTool` 会负责**为每个文件启动一个 `FrontendAction`**，运行你的消费者逻辑。


##### `clang::ASTFrontendAction`

`clang::ASTFrontendAction` 是一个专门用于对源代码进行语法和语义分析（即生成并遍历 AST）的 `FrontendAction`，是你自定义工具逻辑的**容器**或**入口点**。

###### 主要职责

`ASTFrontendAction` 的核心工作只有一个：创建一个 `clang::ASTConsumer` 对象，而 `ASTConsumer` 才是你编写实际工具逻辑的地方。



##### `clang::FrontendAction`

> [!info] Clang 的编译流程
> 当 Clang 编译一个源文件时，它会执行一系列步骤：预处理、词法分析、语法分析、语义分析（生成 AST）、生成 LLVM IR、优化、生成机器码等。

`clang::FrontendAction` 这个类代表了一个可以被 Clang 前端执行的动作，这个动作并不一定是生成目标代码。

Clang 可以被驱动来执行不同的动作，例如：
- `EmitObjAction`: 生成目标文件（`.o`）的默认动作。
- `ASTDumpAction`: 打印 AST 信息（类似于 `clang -Xclang -ast-dump`）。
- `SyntaxOnlyAction`: 只检查语法和语义，不生成代码。
- `ASTFrontendAction`: 一个专门为自定义 AST 分析而设计的动作。



##### `clang::ASTConsumer`

`clang::ASTConsumer` 是一个接口类，它定义了一系列方法（hook），Clang 的前端会在 AST 生成过程的不同阶段调用这些方法，将 AST 的各个部分喂给你的工具。

可以把它想象成一个消费者：Clang 前端是生产者，负责解析代码并生产出 AST；而 `ASTConsumer` 就是消费者，负责消费（即处理和分析）这些 AST。

> [!info] 在编译流程中的角色
>  Clang 处理一个源文件的简化流程如下：
>  1. **预处理**： 处理 `#include`, `#define` 等。
>  2. **词法分析**： 将字符流转换为令牌（Token）流。
>  3. **语法分析**： 根据语法规则构建 AST 节点。
>  4. **语义分析**： 填充符号表，进行类型检查，完成 AST 的构建。
>  5. **`ASTConsumer` 调用**：前端将完全构建好的、经过语义检查的 AST 传递给 `ASTConsumer` 的各个方法。
>  6. (后续可能还有生成 IR、优化、生成代码等步骤，但对于分析工具来说，到第 5 步就结束了)。

###### 常用方法（hook）

`ASTConsumer` 提供了多个虚函数，你可以通过重写它们来注入你的逻辑，其中最重要的两个是：
1. `HandleTopLevelDecl(DeclGroupRef DGR)`：
	- 每当解析到一个顶层的声明（如函数、变量、类、命名空间等）时，前端就会调用此方法。
	- 它是在 AST 构建过程中**增量式调用**的，而不是在最后一次性调用。
	- 适用于一些简单的、不需要看到整个 AST 就能进行的处理。
2. `HandleTranslationUnit(ASTContext &Ctx)`：
	- （**最重要、最常用**）当整个翻译单元（即一个源文件）的 AST 被完全构建好后，前端会调用此方法。
	- 参数 `ASTContext &Ctx` 是黄金入口，它包含了整个 AST 的所有信息，是获取 AST 中各种节点（如声明、类型、语句）的核心接口。
	- **绝大多数静态分析工具的逻辑都从这里开始**，在这里，你可以使用 `RecursiveASTVisitor` 遍历整个 AST，或者使用 `MatchFinder` 来匹配特定的 AST 模式。
3. `HandleTagDeclDefinition`
4. `HandleInlineFunctionDefinition`
5. `Initialize`
6. `HandleImplicitImport`



##### `clang::RecursiveASTVisitor`

`clang::RecursiveASTVisitor` 是一个用于遍历 Clang AST 的递归访问器类模板，用于以深度优先的顺序遍历整个抽象语法树（AST）。

`clang::RecursiveASTVisitor` 提供了一个框架，可以自动地、递归地访问 AST 中的几乎所有节点，可以通过继承这个类并重写你感兴趣的节点类型的处理方法（例如 `VisitFunctionDecl`）来注入自定义逻辑。

###### 工作流程

1. 进入一个节点（例如一个 `ForStmt` - for 循环语句）。
2. 在访问其子节点之前，调用 `TraverseForStmt(ForStmt*)`。
3. `TraverseForStmt` 会依次调用：
    - `WalkUpFromForStmt(ForStmt*)` （可选，通常用于访问父类）
    - `VisitForStmt(ForStmt*)` — **这就是你可以重写的方法！**
4. 如果 `VisitForStmt` 返回 `true`（默认），访问器会继续递归地遍历这个 for 语句的所有子节点（初始化语句、条件、增量表达式、循环体）。
5. 如果 `VisitForStmt` 返回 `false`，访问器将跳过这个 for 语句所有子节点的遍历。
6. 在所有子节点都被访问完毕后，访问器会继续下一个兄弟节点的访问。

###### 使用

1. **自定义访问器类**：创建一个类，公开继承自 `clang::RecursiveASTVisitor<YourVisitorClass>`
2. **重写访问方法**：重写你关心的节点类型的 `VisitXXX` 方法
	- **常见访问方法**
		- `bool VisitCXXRecordDecl(CXXRecordDecl *)` - 类/结构体/联合体声明或定义
		- `bool VisitClassTemplateDecl(ClassTemplateDecl *)` - 类模板声明
		- `bool VisitFunctionDecl(FunctionDecl *)` - 函数声明或定义
		- `bool VisitCXXMethodDecl(CXXMethodDecl *)` - 类成员函数
		- `bool VisitCXXConstructorDecl(CXXConstructorDecl *)` - 构造函数
		- `bool VisitCXXDestructorDecl(CXXDestructorDecl *)` - 析构函数
		- `bool VisitFunctionTemplateDecl(FunctionTemplateDecl *)` - 函数模板
3. **启动遍历**：在你的 `ASTConsumer::HandleTranslationUnit` 方法中，创建访问器实例并对 AST 的根节点（`TranslationUnitDecl`）调用 `.TraverseDecl(Context.getTranslationUnitDecl())`

> [!attention] 访问方法有非常严格的格式要求
> 1. **返回类型必须为 `bool`**：返回值用来控制遍历行为
> 	- `return true;`：表示继续正常遍历，访问器会继续递归地访问当前节点的子节点。
> 	- `return false;`：表示停止在此分支的遍历，访问器将跳过当前节点的所有子节点，直接转到下一个兄弟节点。
> 2. **函数名称必须精确是 Visit + AST 节点类名**：如 `VisitCXXRecordDecl`
> 3. **参数类型必须是指向特定 AST 节点类型的指针**：如 `clang::CXXRecordDecl *`
> 	- 参数名可以自定义。
> 	- 这个指针指向当前正在被访问的 AST 节点对象，可以通过它获取该类的所有信息：名称、位置、成员、基类、方法等。
> 4. **访问级别是 `public`** ：`RecursiveASTVisitor` 基类会在外部调用它。




#### 基本步骤

1. 创建一个 `RecursiveASTVisitor` 的子类，重写你关心的节点的 `VisitXXX` 方法。
2. 创建一个 `ASTConsumer` 的子类，在其 `HandleTranslationUnit` 方法中启动你的 Visitor 来遍历 AST。
3. 创建一个 `ASTFrontendAction` 的子类，在其 `CreateASTConsumer` 方法中返回你上面创建的 Consumer。
4. 在 `main` 函数中，使用 `ClangTool` 和 `CommonOptionsParser` 来解析命令行参数并运行你的 Action。


#### 示例

```cpp
// MyTool.cpp
#include "clang/AST/ASTConsumer.h"
#include "clang/AST/RecursiveASTVisitor.h"
#include "clang/Frontend/CompilerInstance.h"
#include "clang/Frontend/FrontendAction.h"
#include "clang/Tooling/CommonOptionsParser.h"
#include "clang/Tooling/Tooling.h"
#include "llvm/Support/CommandLine.h"

using namespace clang;
using namespace clang::tooling;

// 1. 创建Visitor
class MyASTVisitor : public RecursiveASTVisitor<MyASTVisitor> {
public:
  bool VisitFunctionDecl(FunctionDecl *FD) {
    llvm::outs() << "Found function: " << FD->getName() << "\n";
    return true; // 继续遍历
  }
};

// 2. 创建Consumer
class MyASTConsumer : public ASTConsumer {
public:
  void HandleTranslationUnit(ASTContext &Context) override {
    Visitor.TraverseDecl(Context.getTranslationUnitDecl());
  }
private:
  MyASTVisitor Visitor;
};

// 3. 创建Action
class MyFrontendAction : public ASTFrontendAction {
public:
  std::unique_ptr<ASTConsumer> CreateASTConsumer(CompilerInstance &CI,
                                                 StringRef file) override {
    return std::make_unique<MyASTConsumer>();
  }
};

// 4. 设置命令行选项
static llvm::cl::OptionCategory MyToolCategory("my-tool options");

int main(int argc, const char **argv) {
  auto expectedParser = CommonOptionsParser::create(argc, argv, MyToolCategory);
  if (!expectedParser) {
    return 1;
  }

  CommonOptionsParser &optionParser = expectedParser.get();
  ClangTool Tool(optionParser.getCompilations(), optionParser.getSourcePathList());

  return Tool.run(newFrontendActionFactory<MyFrontendAction>().get());
}
```

**工作流程**：
1. **解析命令行参数**：使用 `CommonOptionsParser` 来解析命令行参数。
	- 调用 `create` 方法，传入 `argc`（参数计数）、`argv`（参数向量）和 `MyToolCategory`（自定义选项类别）
	- `create` 方法返回一个 `Expected` 类型的对象：
		- 如果失败，则程序异常退出。
		- 如果成功，通过 `expectedParser.get()` 获取到 `CommonOptionsParser` 的引用。
2. **配置编译环境**：使用 `optionParser` 中的编译数据库（Compilations）和源文件路径列表（SourcePathList）来初始化 `ClangTool` 对象。
	- `Compilations` 通常包含了编译命令（如 `compile_commands.json` 中的信息）
	- `SourcePathList` 是要处理的源文件列表。
3. **启动 `ClangTool::run`**，对每个源文件：
    - 在 `ClangTool` 中为这个文件创建前端动作 `MyFrontendAction` 实例。
    - Clang 调用 `MyFrontendAction::CreateASTConsumer` 方法创建并返回了 `MyASTConsumer` 示例。
    - Clang 前端解析文件，完成所有语义分析，构建出完整的 AST。
    - Clang 调用 `MyASTConsumer::HandleTranslationUnit(ASTContext &Ctx)`，并将整个 AST 的上下文传递进来。
	- 在 `MyASTConsumer::HandleTranslationUnit(ASTContext &Ctx)` 方法中调用了 `visitor.TraverseDecl(Context.getTranslationUnitDecl())`，这里的 `getTranslationUnitDecl()` 就是整个 AST 的根节点，**`MyASTVisitor` 启动**。
	- 当 `MyASTVisitor` 在遍历过程中遇到一个 `CXXRecordDecl` 节点（即一个 `class`/`struct`/`union` 的定义或声明）时，它就会**自动调用重写的 `VisitCXXRecordDecl` 方法**。
    - 输出分析结果


### 使用 Clang C API

C 语言接口，稳定的 ABI，适合长期维护的工具



### 使用 AST Matchers

AST Matchers 是一个声明式的、领域特定语言（DSL），允许你像写查询语句一样来描述你感兴趣的 AST 节点模式。

#### 核心概念

- **Matcher**：一个函数，用于描述节点类型和其属性。例如：
    - `functionDecl()`：匹配任何函数声明。
    - `functionDecl(hasName("main"))`：匹配名为 “main” 的函数声明。
    - `callExpr(callee(functionDecl(hasName("printf"))))`：匹配所有调用 `printf` 函数的表达式。

#### 使用步骤

1. 包含 `clang/ASTMatchers/ASTMatchers.h` 和 `clang/ASTMatchers/ASTMatchFinder.h`。
2. 创建一个 `MatchFinder` 对象。
3. 使用 `MatchFinder::addMatcher` 为你关心的模式添加匹配器，并关联一个回调函数。
4. 运行你的工具。


#### 示例

```cpp
#include ...
using namespace clang;
using namespace clang::ast_matchers;

// 回调函数，当匹配成功时调用
class MyMatchCallback : public MatchFinder::MatchCallback {
public:
  virtual void run(const MatchFinder::MatchResult &Result) override {
    if (const auto *CE = Result.Nodes.getNodeAs<CallExpr>("call")) {
      CE->dump(); // 打印出这个调用表达式的AST信息
    }
  }
};

int main(int argc, const char **argv) {
  ...
  MyMatchCallback Callback;
  MatchFinder Finder;

  // 定义匹配器：查找所有调用表达式，其被调用的函数名为 "printf"
  StatementMatcher printfMatcher = callExpr(callee(functionDecl(hasName("printf")))).bind("call");

  Finder.addMatcher(printfMatcher, &Callback);

  // 运行Tool，Finder将负责在AST中寻找匹配的节点
  std::unique_ptr<FrontendActionFactory> Factory =
      newFrontendActionFactory(&Finder);
  return Tool.run(Factory.get());
}
```


## 应用场景

1. **静态代码分析器**（如 Clang-Tidy）：查找代码中的 bug、编码风格违规、性能问题等。
2. **重构工具**（如 Clang Modernizer）：自动化代码转换，例如将旧式 C++ 代码升级到 C++11/14/17。
3. **代码格式化工具**（如 ClangFormat）：理解代码结构以进行格式化。
4. **生成文档**：从源代码注释和结构中提取信息。
5. **创建自定义语言插件**。



