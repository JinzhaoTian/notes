Unreal Header Tool（UHT，虚幻头工具） 是一个预编译器，它在常规 C++ 编译器之前运行，解析头文件中的元数据宏，并生成额外的反射代码（主要是 `*.generated.h` 和 `*.gen.cpp` 文件）。

> [!tip]
> 基于 C++ 编写的 UHT 在 UE 5.2 之后被放弃了，转到基于 C# 编写的 UHT 了，在 `EpicGames.UHT.dll` 这个模块里，然后在 UBT 中调用。

## 工作流程

1. **扫描项目源代码**：UHT 会解析所有头文件（`.h`），查找包含 `GENERATED_BODY()` 宏以及 `UCLASS()`, `UPROPERTY()` 等反射宏的类。
2. **解析宏与提取元数据**：UHT 并非真正的 C++ 语法分析器，但它理解 Unreal 的反射宏语法。它会提取出类名、父类、属性类型、函数签名、元数据说明符（如 `EditAnywhere`, `BlueprintCallable`）等信息。
3. **生成反射代码**：根据提取的信息，UHT 会为每个标记的类生成大量的模板代码，这些代码主要包括：
    - **`GetPrivateStaticClass()` 函数**：用于获取该类的 `UClass*` 静态实例5。
    - **静态类注册函数**：例如 `StaticClass()`，用于在引擎启动时向核心 `UObject` 系统注册这个类。
    - **属性元数据表**：一个存储该类所有属性（`UProperty`）元数据的结构体，包括属性名称、偏移量、 flags 等。
    - **函数元数据表**：存储该类所有函数（`UFunction`）的元数据，包括函数名称、参数列表、 flags 等。
    - **实现 `UObject` 接口**：生成序列化（`Serialize`）、垃圾回收标记（`AddReferencedObjects`）、网络复制（`ReplicatedNotify`）等函数的默认实现或辅助代码。



## 代码示例


### 创建头文件（`*.h`）

```cpp
// 文件名: MyObject.h

#pragma once

#include "CoreMinimal.h"   // 包含UE核心基础类型和宏
#include "UObject/Object.h" // 如果需要明确继承UObject，虽然CoreMinimal有时已包含，但明确包含更安全
#include "MyObject.generated.h" // 必须包含！UHT将生成反射代码到此文件

UCLASS(Blueprintable, BlueprintType) // UCLASS宏声明反射类，Blueprintable表示可在蓝图中继承，BlueprintType表示可作为蓝图变量类型
class MYPROJECT_API UMyObject : public UObject // 类名前缀通常用"U"，MYPROJECT_API根据你的项目模块命名
{
    GENERATED_BODY() // 必须！GENERATED_BODY()宏替代了类体，用于注入UHT生成的代码

public:
    // 默认构造函数（可选，但建议有）
    UMyObject();

    // 一个可编辑、蓝图可读写的属性
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "MyObject Properties")
    float ExampleFloatProperty;

    // 一个仅蓝图可读的属性
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "MyObject Properties")
    int32 ExampleIntProperty;

    // 一个蓝图可调用的函数
    UFUNCTION(BlueprintCallable, Category = "MyObject Functions")
    void ExampleBlueprintCallableFunction();

    // 一个蓝图可调用且有返回值的函数
    UFUNCTION(BlueprintCallable, Category = "MyObject Functions")
    int32 ExampleFunctionWithReturnValue();

    // 一个可作为蓝图事件实现的函数（C++提供默认实现，蓝图可覆盖）
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "MyObject Functions")
    void ExampleBlueprintNativeEvent();
    
    // 注意：BlueprintNativeEvent函数需要在cpp文件中有一个_Implementation实现体
};
```

### 创建源文件（`*.cpp`）

```cpp
// 文件名: MyObject.cpp

#include "MyObject.h"

// 实现构造函数
UMyObject::UMyObject()
{
    // 初始化默认值
    ExampleFloatProperty = 0.0f;
    ExampleIntProperty = 0;
}

// 实现普通函数
void UMyObject::ExampleBlueprintCallableFunction()
{
    // 在这里实现你的函数逻辑
    UE_LOG(LogTemp, Warning, TEXT("ExampleBlueprintCallableFunction called!"));
}

// 实现带返回值的函数
int32 UMyObject::ExampleFunctionWithReturnValue()
{
    // 在这里实现你的函数逻辑并返回值
    return ExampleIntProperty * 2;
}

// 实现BlueprintNativeEvent函数的默认C++实现
// 函数名是：函数名+_Implementation
void UMyObject::ExampleBlueprintNativeEvent_Implementation()
{
    // 在这里实现你的默认逻辑
    UE_LOG(LogTemp, Warning, TEXT("ExampleBlueprintNativeEvent_Implementation (C++ default) called!"));
}
```


### 生成头文件（`*.generated.h`）

```cpp
/*===========================================================================
Generated code exported from UnrealHeaderTool.
DO NOT modify this manually! Edit the corresponding .h files instead!
===========================================================================*/

#pragma once

#include "UObject/GeneratedCppIncludes.h"
#include "MyObject.h"
PRAGMA_DISABLE_DEPRECATION_WARNINGS
#ifdef MYPROJECT_UMyObject_generated_h
#error "UMyObject.generated.h already included, missing '#pragma once' in UMyObject.h"
#endif
#define MYPROJECT_UMyObject_generated_h

#define MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_GENERATED_BODY \
    PRAGMA_DISABLE_DEPRECATION_WARNINGS \
public: \
    MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_PRIVATE_PROPERTY_OFFSET \
    MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_SPARSE_DATA \
    MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_RPC_WRAPPERS \
    MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_RPC_WRAPPERS_NO_PURE_DECLS \
    MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_INCLASS \
    MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_ENHANCED_CONSTRUCTORS \
    PRAGMA_ENABLE_DEPRECATION_WARNINGS


#define MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_PRIVATE_PROPERTY_OFFSET \
    FORCEINLINE static uint32 __PPO__ExampleFloatProperty() { return STRUCT_OFFSET(UMyObject, ExampleFloatProperty); } \
    FORCEINLINE static uint32 __PPO__ExampleIntProperty() { return STRUCT_OFFSET(UMyObject, ExampleIntProperty); }


#define MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_SPARSE_DATA


#define MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_RPC_WRAPPERS \
 \
    DECLARE_FUNCTION(execExampleBlueprintCallableFunction); \
    DECLARE_FUNCTION(execExampleFunctionWithReturnValue); \
    DECLARE_FUNCTION(execExampleBlueprintNativeEvent);


#define MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_RPC_WRAPPERS_NO_PURE_DECLS \
 \
    DECLARE_FUNCTION(execExampleBlueprintCallableFunction); \
    DECLARE_FUNCTION(execExampleFunctionWithReturnValue); \
    DECLARE_FUNCTION(execExampleBlueprintNativeEvent);


#define MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_INCLASS \
private: \
    static void StaticRegisterNativesUMyObject(); \
    friend struct Z_Construct_UClass_UMyObject_Statics; \
public: \
    DECLARE_CLASS(UMyObject, UObject, COMPILED_IN_FLAGS(CLASS_Edit | CLASS_Config), CASTCLASS_None, TEXT("/Script/MyProject"), NO_API) \
    DECLARE_SERIALIZER(UMyObject) \
    /** Indicates whether the class is compiled into the engine */ \
    enum {IsIntrinsic=COMPILED_IN_INTRINSIC};


#define MYPROJECT_MyObject_Source_MyProject_MyObject_h_14_ENHANCED_CONSTRUCTORS \
    /** Standard constructor, called after all reflected properties have been initialized */ \
    NO_API UMyObject(const FObjectInitializer& ObjectInitializer = FObjectInitializer::Get()); \
private: \
    /** Private move- and copy-constructors, should never be used */ \
    NO_API UMyObject(UMyObject&&); \
    NO_API UMyObject(const UMyObject&); \
public: \
    DECLARE_VTABLE_PTR_HELPER_CTOR(NO_API, UMyObject); \
    DEFINE_VTABLE_PTR_HELPER_CTOR_CALLER(UMyObject); \
    DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL(UMyObject)


template<> MYPROJECT_API UClass* StaticClass<class UMyObject>();

#undef CURRENT_FILE_ID
#define CURRENT_FILE_ID MYPROJECT_MyObject_Source_MyProject_MyObject_h_14


PRAGMA_ENABLE_DEPRECATION_WARNINGS
```

### 生成源代码（`*.gen.cpp`）

```cpp
// 此文件由 Unreal Header Tool 自动生成
// 请不要手动修改

#include "UObject/GeneratedCppIncludes.h"
#include "MyProject/MyObject.h"
PRAGMA_DISABLE_DEPRECATION_WARNINGS
void EmptyLinkFunctionForGeneratedCodeMyObject() {}
// Cross Module References
    MYPROJECT_API UClass* Z_Construct_UClass_UMyObject();
    MYPROJECT_API UClass* Z_Construct_UClass_UMyObject_NoRegister();
    MYPROJECT_API UClass* Z_Construct_UClass_UObject();
    UPackage* Z_Construct_UPackage__Script_MyProject();
// End Cross Module References
    DEFINE_FUNCTION(UMyObject::execExampleBlueprintCallableFunction)
    {
        P_FINISH;
        P_NATIVE_BEGIN;
        P_THIS->ExampleBlueprintCallableFunction();
        P_NATIVE_END;
    }
    DEFINE_FUNCTION(UMyObject::execExampleFunctionWithReturnValue)
    {
        P_FINISH;
        P_NATIVE_BEGIN;
        *(int32*)Z_Param__Result=P_THIS->ExampleFunctionWithReturnValue();
        P_NATIVE_END;
    }
    DEFINE_FUNCTION(UMyObject::execExampleBlueprintNativeEvent)
    {
        P_FINISH;
        P_NATIVE_BEGIN;
        P_THIS->ExampleBlueprintNativeEvent_Implementation();
        P_NATIVE_END;
    }
    void UMyObject::StaticRegisterNativesUMyObject()
    {
        UClass* Class = UMyObject::StaticClass();
        static const FNameNativePtrPair Funcs[] = {
            { "ExampleBlueprintCallableFunction", &UMyObject::execExampleBlueprintCallableFunction },
            { "ExampleFunctionWithReturnValue", &UMyObject::execExampleFunctionWithReturnValue },
            { "ExampleBlueprintNativeEvent", &UMyObject::execExampleBlueprintNativeEvent },
        };
        FNativeFunctionRegistrar::RegisterFunctions(Class, Funcs, UE_ARRAY_COUNT(Funcs));
    }
    struct Z_Construct_UClass_UMyObject_Statics
    {
        static UObject* (*const DependentSingletons[])();
        static const FClassFunctionLinkInfo FuncInfo[];
#if WITH_METADATA
        static const UECodeGen_Private::FMetaDataPairParam Class_MetaDataParams[];
#endif
#if WITH_METADATA
        static const UECodeGen_Private::FMetaDataPairParam NewProp_ExampleFloatProperty_MetaData[];
#endif
        static const UECodeGen_Private::FFloatPropertyParams NewProp_ExampleFloatProperty;
#if WITH_METADATA
        static const UECodeGen_Private::FMetaDataPairParam NewProp_ExampleIntProperty_MetaData[];
#endif
        static const UECodeGen_Private::FIntPropertyParams NewProp_ExampleIntProperty;
        static const UECodeGen_Private::FPropertyParamsBase* const PropPointers[];
        static const FCppClassTypeInfoStatic StaticCppClassTypeInfo;
        static const UECodeGen_Private::FClassParams ClassParams;
    };
    UObject* (*const Z_Construct_UClass_UMyObject_Statics::DependentSingletons[])() = {
        (UObject* (*)())Z_Construct_UClass_UObject,
        (UObject* (*)())Z_Construct_UPackage__Script_MyProject,
    };
    const FClassFunctionLinkInfo Z_Construct_UClass_UMyObject_Statics::FuncInfo[] = {
        { &UMyObject::execExampleBlueprintCallableFunction, "ExampleBlueprintCallableFunction" }, // 1169361420
        { &UMyObject::execExampleFunctionWithReturnValue, "ExampleFunctionWithReturnValue" }, // 3491849730
        { &UMyObject::execExampleBlueprintNativeEvent, "ExampleBlueprintNativeEvent" }, // 198048298
    };
#if WITH_METADATA
    const UECodeGen_Private::FMetaDataPairParam Z_Construct_UClass_UMyObject_Statics::Class_MetaDataParams[] = {
        { "BlueprintType", "true" },
        { "IncludePath", "MyObject.h" },
        { "ModuleRelativePath", "MyObject.h" },
        { "Blueprintable", "true" },
    };
#endif
#if WITH_METADATA
    const UECodeGen_Private::FMetaDataPairParam Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleFloatProperty_MetaData[] = {
        { "Category", "MyObject Properties" },
        { "ModuleRelativePath", "MyObject.h" },
    };
#endif
    const UECodeGen_Private::FFloatPropertyParams Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleFloatProperty = {
        "ExampleFloatProperty",
        nullptr,
        (EPropertyFlags)0x0010000000000005,
        UECodeGen_Private::EPropertyGenFlags::Float,
        RF_Public|RF_Transient|RF_MarkAsNative,
        1,
        STRUCT_OFFSET(UMyObject, ExampleFloatProperty),
        METADATA_PARAMS(Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleFloatProperty_MetaData, UE_ARRAY_COUNT(Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleFloatProperty_MetaData))
    };
#if WITH_METADATA
    const UECodeGen_Private::FMetaDataPairParam Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleIntProperty_MetaData[] = {
        { "Category", "MyObject Properties" },
        { "ModuleRelativePath", "MyObject.h" },
        { "ToolTip", "一个仅蓝图可读的属性" },
    };
#endif
    const UECodeGen_Private::FIntPropertyParams Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleIntProperty = {
        "ExampleIntProperty",
        nullptr,
        (EPropertyFlags)0x0010000000000004,
        UECodeGen_Private::EPropertyGenFlags::Int,
        RF_Public|RF_Transient|RF_MarkAsNative,
        1,
        STRUCT_OFFSET(UMyObject, ExampleIntProperty),
        METADATA_PARAMS(Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleIntProperty_MetaData, UE_ARRAY_COUNT(Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleIntProperty_MetaData))
    };
    const UECodeGen_Private::FPropertyParamsBase* const Z_Construct_UClass_UMyObject_Statics::PropPointers[] = {
        (const UECodeGen_Private::FPropertyParamsBase*)&Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleFloatProperty,
        (const UECodeGen_Private::FPropertyParamsBase*)&Z_Construct_UClass_UMyObject_Statics::NewProp_ExampleIntProperty,
    };
    const FCppClassTypeInfoStatic Z_Construct_UClass_UMyObject_Statics::StaticCppClassTypeInfo = {
        TCppClassTypeTraits<UMyObject>::IsAbstract,
    };
    const UECodeGen_Private::FClassParams Z_Construct_UClass_UMyObject_Statics::ClassParams = {
        &UMyObject::StaticClass,
        nullptr,
        &StaticCppClassTypeInfo,
        DependentSingletons,
        FuncInfo,
        Z_Construct_UClass_UMyObject_Statics::PropPointers,
        nullptr,
        UE_ARRAY_COUNT(DependentSingletons),
        UE_ARRAY_COUNT(FuncInfo),
        UE_ARRAY_COUNT(Z_Construct_UClass_UMyObject_Statics::PropPointers),
        0,
        0x009000A0u,
        METADATA_PARAMS(Z_Construct_UClass_UMyObject_Statics::Class_MetaDataParams, UE_ARRAY_COUNT(Z_Construct_UClass_UMyObject_Statics::Class_MetaDataParams))
    };
    UClass* Z_Construct_UClass_UMyObject()
    {
        if (!Z_Registration_Info_UClass_UMyObject.OuterSingleton)
        {
            UECodeGen_Private::ConstructUClass(Z_Registration_Info_UClass_UMyObject.OuterSingleton, Z_Construct_UClass_UMyObject_Statics::ClassParams);
        }
        return Z_Registration_Info_UClass_UMyObject.OuterSingleton;
    }
    template<> MYPROJECT_API UClass* StaticClass<UMyObject>()
    {
        return UMyObject::StaticClass();
    }
    DEFINE_VTABLE_PTR_HELPER_CTOR(UMyObject);
    UMyObject::UMyObject(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer) {
        // ... 属性初始化代码
    }
    PRAGMA_ENABLE_DEPRECATION_WARNINGS
```


## 解析

最新的 Unreal Header Tool 使用 .NET 8.0 进行了重构：
- Path：`/Engine/Source/Programs/Shared/EpicGames.UHT`

`EpicGames.UHT` 并没有使用专门的 C++ AST 解析库（如 Clang），而是实现了自己的文本解析系统。

### 解析流程

1. **源文件读取**
```csharp
// UhtSourceFile.cs - 直接读取文件为字符串
public StringView Data => _sourceFragment.Data;
public virtual void Read()
{
    _sourceFragment = Session.ReadSource(FilePath);
}
```

2. **词法分析器 (Tokenizer)**：UHT 实现了完整的词法分析系统，将 C++ 源码分解为标记(Token)：
```csharp
// UhtToken.cs - 定义各种文本标记类型
public enum UhtTokenType
{
    EndOfFile,
    Identifier,      // 标识符
    Symbol,          // 符号 
    FloatConst,      // 浮点常量
    DecimalConst,    // 十进制常量
    StringConst,     // 字符串常量
    // ...
}
```

3. **基于字符的解析**
```csharp
// UhtTokenBufferReader.cs - 对字符数据进行逐字符扫描
public UhtTokenBufferReader(IUhtMessageSite messageSite, ReadOnlyMemory<char> input)
{
    _data = input; // 直接处理字符数据
}
```

4. **手工实现的语法分析**：各种解析器都是基于文本标记进行语法分析：
```csharp
// UhtClassParser.cs - 手工解析 UCLASS 宏
private static UhtParseResult ParseUClass(UhtParsingScope parentScope, ref UhtToken token)
{
    // 解析类定义的文本结构
    topScope.TokenReader.Require("class");
    classObj.SourceName = topScope.TokenReader.GetIdentifier().Value.ToString();
}
```

### 工作流程

1. **文件读取**: 将 C++ 头文件作为纯文本读入内存
2. **词法分析**: 逐字符扫描，识别标识符、关键字、符号、常量等
3. **预处理**: 处理 `#include`、`#define` 等预处理指令
4. **语法分析**: 基于标记序列识别 UE 宏（`UCLASS`、`UFUNCTION` 等）
5. **构建语法树**: 将解析结果构建为内部的类型系统

### 为什么选择纯文本解析？

**优点**：
1. **轻量级**: 不依赖复杂的 C++ 解析库
2. **可控性**: 完全控制解析逻辑，可以专门优化 UE 宏处理
3. **速度**: 针对 UHT 需求优化的解析器通常比通用解析器更快
4. **简单**: 只需要解析 UE 特定的宏，不需要完整的 C++ 语法支持

**局限性**：
1. **有限的 C++ 支持**: 只能处理特定的 C++ 语法结构
2. **维护成本**: 需要自己维护词法和语法分析器
3. **容错性**: 对复杂 C++ 语法的处理能力有限
