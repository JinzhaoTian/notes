gtest 是由 Google 开发和维护的一个广泛使用的 C++ 单元测试框架，为 C++ 开发者提供了一套丰富而强大的工具，用于编写、组织和运行自动化测试，从而确保代码的正确性和稳定性。

## 主要特点

1. **丰富的断言**：提供了多种断言（Assertions）用于验证测试条件，包括 `ASSERT_*` 和 `EXPECT_*` 系列，如 `ASSERT_EQ`、`EXPECT_TRUE` 等。
2. **测试夹具（Test Fixtures）**：支持测试夹具，允许在多个测试中共享相同的设置和清理代码，提高代码复用性。
3. **参数化测试**：支持参数化测试，可以使用不同的输入数据集多次运行同一个测试案例，方便测试不同的输入组合。
4. **类型参数化测试**：允许对多种类型进行相同的测试，适用于模板类和函数的测试。
5. **死亡测试（Death Tests）**：支持测试程序在特定条件下的崩溃行为，例如异常抛出或断言失败。
6. **多线程支持**：支持在多线程环境中运行测试，确保线程安全性。
7. **良好的集成性**：与多种构建系统（如 CMake）和持续集成工具（如 Jenkins、GitHub Actions）无缝集成。
8. **详细的报告**：生成详细的测试结果报告，便于调试和分析。


## 优势

1. **成熟稳定**：经过多年发展，Google Test 已经非常稳定，并被广泛应用于各种大型项目中。
2. **社区支持**：拥有活跃的社区和丰富的文档资源，开发者可以轻松获取帮助和示例。
3. **扩展性强**：支持自定义断言和事件监听器，允许开发者根据需要扩展功能。
4. **跨平台**：支持多种操作系统，包括 Windows、Linux 和 macOS，适用于跨平台开发。


## 最佳实践

1. **组织测试代码**：将测试代码与源代码分开，通常放在独立的 `test` 目录中。
2. **持续集成**：结合 CI 工具（如 GitHub Actions、GitLab CI）自动运行测试，确保代码的持续健康。
3. **覆盖率分析**：使用代码覆盖率工具（如 gcov）与 Google Test 结合，评估测试覆盖率，发现潜在的未测试代码。
4. **保持测试独立**：确保每个测试用例独立运行，避免相互依赖，提高测试的可靠性和可维护性。



## 安装与集成

通过 CMake 集成 Google Test

1. **添加子模块**（如果使用 Git）：
```bash
git submodule add https://github.com/google/googletest.git
git submodule update --init
```

2. **在 `CMakeLists.txt` 中配置**：
```c++
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 添加 Google Test
add_subdirectory(googletest)

# 添加测试可执行文件
enable_testing()
add_executable(my_tests test/test_main.cpp)

# 链接 Google Test 库
target_link_libraries(my_tests gtest gtest_main)

# 添加测试
add_test(NAME MyTests COMMAND my_tests)
```

3. **编写测试代码**（例如 `test/test_main.cpp`）
```c++
#include <gtest/gtest.h>

// 示例测试案例
TEST(SampleTest, AssertionTrue) {
    ASSERT_TRUE(1 == 1);
}

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

4. **编译并运行测试**：
```c++
mkdir build && cd build
cmake ..
make
./my_tests
```


## 使用示例

以下是一些常用的 Google Test 功能示例：

**基本断言**

```c++
#include <gtest/gtest.h>

TEST(MathTest, Addition) {
    EXPECT_EQ(2 + 2, 4);         // 期望 2 + 2 等于 4
    ASSERT_NE(2 * 2, 5);         // 断言 2 * 2 不等于 5
}
```

**测试夹具**

```c++
#include <gtest/gtest.h>

class MyTestFixture : public ::testing::Test {
protected:
    void SetUp() override {
        // 测试前的初始化代码
    }

    void TearDown() override {
        // 测试后的清理代码
    }

    // 共享的测试数据
    int shared_value;
};

TEST_F(MyTestFixture, Test1) {
    shared_value = 5;
    EXPECT_EQ(shared_value, 5);
}

TEST_F(MyTestFixture, Test2) {
    shared_value = 10;
    EXPECT_EQ(shared_value, 10);
}
```

**参数化测试**

```c++
#include <gtest/gtest.h>

// 定义测试参数类型
struct TestParam {
    int input;
    int expected;
};

// 创建参数化测试夹具
class ParamTest : public ::testing::TestWithParam<TestParam> {};

TEST_P(ParamTest, HandlesVariousInputs) {
    TestParam param = GetParam();
    EXPECT_EQ(param.input * 2, param.expected);
}

// 定义测试参数
INSTANTIATE_TEST_SUITE_P(
    MultiplyTests,
    ParamTest,
    ::testing::Values(
        TestParam{1, 2},
        TestParam{2, 4},
        TestParam{3, 6}
    )
);
```