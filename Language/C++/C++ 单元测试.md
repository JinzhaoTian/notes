
## 单元测试框架

### 1. Google Test (gtest)

- **特点**：Google Test 是目前最受欢迎的 C++ 单元测试框架之一，由 Google 开发并维护。
- **优点**：
    - 支持断言（如 `ASSERT_EQ`、`EXPECT_TRUE` 等）。
    - 能与 CMake 无缝集成。
    - 提供丰富的测试功能（如参数化测试、性能测试等）。
    - 良好的文档和社区支持。
- **用法**：
```C++
#include <gtest/gtest.h>

TEST(SampleTest, AssertionTrue) {
    ASSERT_TRUE(1 == 1);
}

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

### 2. Catch2

- **特点**：Catch2 是一个单头文件的 C++ 测试框架，易于集成和使用。
- **优点**：
    - 单头文件，直接包含到项目中即可使用。
    - 支持简单的 BDD 风格（行为驱动开发）语法。
    - 提供详细的断言输出，方便调试。
- **用法**：
```c++
#define CATCH_CONFIG_MAIN
#include <catch2/catch.hpp>

TEST_CASE("Test Case 1") {
    REQUIRE(1 == 1);
}
```

### 3. CppUnit

- **特点**：CppUnit 是基于 JUnit 的 C++ 测试框架，适合喜欢传统测试框架风格的用户。
- **优点**：
    - 提供类库，支持创建丰富的测试用例和测试套件。
    - 与其他 `xUnit` 测试框架一致的设计。
- **用法**： CppUnit 通常需要更多的配置和更复杂的测试代码风格。
```c++
CPPUNIT_ASSERT_EQUAL(1, 1);
```


### 4. Boost.Test

- **特点**：Boost.Test 是 Boost 库中的一个模块，功能强大，支持多种测试类型。
- **优点**：
    - 集成了异常处理、日志记录和测试用例过滤等高级功能。
    - 与其他 Boost 库兼容，适合使用 Boost 的项目。
- **用法**：
```C++
#define BOOST_TEST_MODULE MyTest
#include <boost/test/included/unit_test.hpp>
BOOST_AUTO_TEST_CASE(test_case_1) {
    BOOST_CHECK(1 == 1);
}
```

### 5. Doctest

- **特点**：Doctest 是轻量级的 C++ 测试框架，设计理念类似于 Catch2。
- **优点**：
    - 单头文件，易于集成。
    - 启动速度快，适合大型项目的快速开发和测试。
    - 语法简单，容易上手。
- **用法**：
```C++
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include <doctest/doctest.h>

TEST_CASE("Test Case 1") {
    CHECK(1 == 1);
}
```

### 6. UnitTest++

- **特点**：UnitTest++ 是一个轻量级的单元测试框架，提供基础的断言和测试功能。
- **优点**：
    - 轻量，易于集成和使用。
    - 代码量小，适合小型项目。
- **用法**：
```C++
#include <unittest++/UnitTest++.h>

TEST(SampleTest) {
    CHECK_EQUAL(1, 1);
}
```





## 单元测试流程

### 1. 设置单元测试

常用的C++单元测试框架包括 **Google Test (gtest)** 和 **Catch2**。下面以 Google Test 为例：

#### 安装 Google Test

你可以通过包管理器或手动方式安装 Google Test。以 CMake 为例，可以通过以下步骤集成 Google Test：

1. **在项目中添加 Google Test 子模块**：
```C++
git submodule add https://github.com/google/googletest.git
git submodule update --init
```

2. **在 `CMakeLists.txt` 中配置 Google Test**： 在项目的 `CMakeLists.txt` 文件中，添加以下内容：
```C++
# GoogleTest requires at least C++11
set(CMAKE_CXX_STANDARD 11)

# Add Google Test
add_subdirectory(googletest)

# Add a test executable
add_executable(my_tests test/test_main.cpp)

# Link Google Test and the main project
target_link_libraries(my_tests gtest gtest_main)
```

3. **编写测试代码**： 在 `test/test_main.cpp` 中编写简单的测试用例：
```C++
#include <gtest/gtest.h>

// A sample test case
TEST(SampleTest, AssertionTrue) {
    ASSERT_TRUE(1 == 1);
}

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```


4. **运行测试**： 在项目根目录下，编译并运行测试：
```c++
mkdir build && cd build
cmake ..
make
./my_tests
```


### 2. 设置 CI/CD 流程

CI/CD（持续集成/持续交付）通过自动化测试、构建和部署，确保代码质量。以 **GitLab CI** 和 **GitHub Actions** 为例。

#### GitLab CI

GitLab CI 使用 `.gitlab-ci.yml` 配置文件来定义流水线。

1. **创建 `.gitlab-ci.yml` 文件**：
```C++
stages:
  - build
  - test

build:
  stage: build
  image: gcc:latest
  script:
    - mkdir build
    - cd build
    - cmake ..
    - make

test:
  stage: test
  image: gcc:latest
  script:
    - cd build
    - ./my_tests
```

这个配置文件分为两个阶段：`build` 和 `test`，它将代码编译成可执行文件并运行测试。
    
2. **提交并运行 CI**： 将 `.gitlab-ci.yml` 文件提交到 GitLab 仓库后，GitLab CI 将自动触发流水线来运行编译和测试。

#### GitHub Actions

GitHub Actions 使用 `.github/workflows/` 下的 YAML 文件来配置流水线。

1. **创建 GitHub Actions 工作流**： 在 `.github/workflows/ci.yml` 中配置：
```C++
name: CI

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up CMake
        uses: jwlawson/actions-setup-cmake@v1

      - name: Build with CMake
        run: |
          mkdir build
          cd build
          cmake ..
          make

      - name: Run tests
        run: |
          cd build
          ./my_tests
```

2. **提交代码**： 推送代码后，GitHub Actions 会自动运行流水线，执行构建和测试。

### 3. 提示

- **扩展测试**：可以根据项目需求，编写更复杂的测试用例，覆盖边界情况和异常处理。
- **持续部署**：根据项目需要，CI/CD 还可以进一步配置持续部署，如将构建结果自动部署到服务器或发布到包管理平台。