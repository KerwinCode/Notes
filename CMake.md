CMake 是一种跨平台的构建系统生成器，可以自动生成用于编译、测试和打包 C/C++ 代码的 Makefile 或者 IDE 项目文件。本教程将为您提供 CMake 的基本使用教程。

## 安装 CMake

首先，您需要安装 CMake。可以从 CMake 的官方网站 https://cmake.org/download/ 下载并安装最新版本。

## 简单的 CMakeLists.txt 文件

CMake 的配置文件被称为 CMakeLists.txt，其中包含了项目的构建信息，如编译选项、链接库等。以下是一个简单的 CMakeLists.txt 文件示例：

```cmake
cmake_minimum_required(VERSION 3.10)

project(MyProject)

add_executable(myapp main.cpp)
```

这个文件定义了一个名为 MyProject 的项目，它包含一个可执行文件 myapp，该文件由 main.cpp 源文件构建而成。

## 基本的构建过程

要构建项目，请执行以下步骤：

1. 在项目根目录下创建一个 build 文件夹（或者其他名称），并进入该目录：

```bash
mkdir build
cd build
```

1. 运行 CMake：

```bash
cmake ..
```

这会在 build 文件夹中生成 Makefile 或者 IDE 项目文件，具体取决于您的配置。

1. 执行 make 命令（或者在 IDE 中编译项目）：

```bash
make
```

这将编译项目并生成可执行文件。

1. 运行可执行文件：

```bash
./myapp
```

## 添加头文件和库

如果您的项目需要使用第三方库，您需要告诉 CMake 该如何找到这些库。假设您的项目需要使用 OpenSSL 库，请按照以下步骤进行操作：

1. 在 CMakeLists.txt 文件中添加以下行：

```cmake
find_package(OpenSSL REQUIRED)
include_directories(${OPENSSL_INCLUDE_DIR})
target_link_libraries(myapp ${OPENSSL_LIBRARIES})
```

这将告诉 CMake 找到 OpenSSL 库并将其链接到 myapp 可执行文件中。

1. 运行 CMake：

```bash
cmake ..
```

1. 编译项目：

```bash
make
```

## 文档

CMake 官方文档 https://cmake.org/documentation/

CMake 中文文档 https://runebook.dev/zh-CN/docs/cmake/-index-CMake 

Modern CMake 简体中文版 https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/

CMake菜谱（CMake Cookbook中文版）https://www.bookstack.cn/read/CMake-Cookbook/README.md

CMakeLearn 项目 https://github.com/SuperH-0630/CMakeLearn

```cmake
#[[==============================================================
文件名: CMakeLists.txt

cmake_minimum_required不仅可以设定一个版本最小值, 还可以设定版本的一个范围, 具体可以参见文档
project是顶级CMakeLists.txt所必须的, Languages表示该项目需要使用的语言, c++语言用CXX表示
add_executable的第一个参数表示可执行程序的名字, 不需要包含后缀, cmake会根据不同的平台设定正确的后缀

add_library与add_executable类型, 用于生成一个函数库(默认情况下是静态库)
target_sources会追加源文件到目标中
目标: 即通过add_librar与add_executable构建的对象, 例如此例子中的msg和hello
为什么.h要写入头文件中: 其实我也没有很确切的理由, 实际上尽管不添加大多数情况也可以运行, 我想应该是文件依赖管理吧
PUBLIC是什么意思: 例如hello依赖msg库, 则hello的源文件中也会自动添加msg的PUBLIC源文件
INTERFACE和PRIVATE: 除了PUBLIC, 还有这两种类型.
    PRIVATE表示只添加到msg中
    INTERFACE表示只添加到依赖msg的目标中(例如hello)
    PUBLIC = PRIVATE + INTERFACE
target_link_libraries用于添加一个库链接

if() ... elseif() ... else() ... endif() 表示一个条件语句
    注意: if的条件访问变量时不需要${}符号, 例如 if(var1)实际上就是条件时var1的值而不是var1字符串本身 (if已经自动解引用变量, 不需要再显式使用${})
option 表示添加一个CMake参数
    注意: 在cmake-gui只有当该option被运算时, 才会将option显示出来
    例如, 将option(BUILD_LIBRARY "Build msg library" ON)修改为option(BUILD_LIBRARY "Build msg library" OFF)
        初次配置CMake时便将不会显示BUILD_SHARED_LIBRARY选项, 因为BUILD_LIBRARY为OFF
        option(BUILD_SHARED_LIBRARY "Build shared library" ON)没有执行
    在命令行则可以直接通过-Dxxx=yyy来使用参数
add_library添加的源码默认为PRIVATE

message 用于在CMake终端输出信息
CMAKE_C_COMPILER等变量记录的是编译器等的信息, 修改这些变量可以对编译产生影响
也可以通过-DCMAKE_C_COMPILER=xxx的方式在命令行修改这些选项

foreach(var item1 item2 item3...) ... endforeach() 表示一个遍历语句
foreach 还有range的迭代方式, 可以参见文档 foreach(var strat [end] [step])

add_subdirectory添加一个子目录 注意: 是添加子目录而不是子程序 (子程序需要使用超级构建)

include 可以导入文件来执行(一般是.cmake文件), 并且不会生成新的变量空间

可以设置CMAKE_ARCHIVE_OUTPUT_DIRECTORY, CMAKE_RUNTIME_OUTPUT_DIRECTORY等变量来控制add_library, add_executable对目标的输出位置

execute_process 在CMake配置时运行程序
add_custom_command 添加自定义命令
add_custom_target 添加自定义目标

file 指令主要处理文件IO相关的操作
list 指令主要处理列表相关的操作
=================================================================]]

cmake_minimum_required(VERSION 3.20)
project(cmake-learn LANGUAGES C)
set(CMAKE_C_STANDARD 11)
set(C_EXTENSIONS OFF)
set(C_STANDARD_REQUIRED OFF)

include(cmake/CMakeFindExternalProject/init.cmake)
wi_set_install_dir()  # 设置安装的目录

cfep_find_dir(msgTargets
              SOURCE_DIR "${CMAKE_SOURCE_DIR}/deps/msg"
              EXTERNAL TIMEOUT 60)

add_executable(learn main.c)
target_link_libraries(learn message::msg)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
find_package(PythonSelf)

if (PythonSelf_FOUND)
    message(STATUS "PythonSelf found.")
    message(STATUS "PythonSelf = ${PythonSelf}")
    message(STATUS "PythonSelf_LIBRARIES = ${PythonSelf_LIBRARIES}")
    message(STATUS "PythonSelf_LIBS = ${PythonSelf_LIBS}")  # FindPythonSelf.cmake 中定义的变量, 在这里可以直接访问
    message(STATUS "Python3_EXECUTABLE = ${Python3_EXECUTABLE}")
else()
    message(STATUS "PythonSelf not found.")
endif()

wi_copy_import(TARGETS message::msg)
if (msgTargets_CFEP_FOUND)  # 拷贝内容
    if (WIN32)  # 只有win32才执行, 因为安装时cfep_install会直接拷贝整个文件夹(包括这里生成的.dll)到安装目录
        file(TOUCH ${msgTargets_CFEP_INSTALL}/bin/test_a.dll)  # 为检验wi_copy_dll_bin创建而创建
        file(TOUCH ${msgTargets_CFEP_INSTALL}/test_a2.dll)  # 为检验wi_copy_dll_dir创建而创建
        file(TOUCH ${msgTargets_CFEP_INSTALL}/test_a2.exe)  # 为检验wi_copy_dll_dir创建而创建
    endif ()

    wi_copy_dll_bin(DIRS ${msgTargets_CFEP_INSTALL}/bin)
    wi_copy_dll_dir(DIRS ${msgTargets_CFEP_INSTALL})
endif()

file(TOUCH ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test.dll)  # 为检验wi_install_dll_bin创建而创建
file(TOUCH ${CMAKE_BINARY_DIR}/test2.dll)  # 为检验wi_install_dll_dir创建而创建
file(TOUCH ${CMAKE_BINARY_DIR}/test2.exe)  # 为检验wi_install_dll_dir创建而创建

wi_install(INSTALL TARGETS learn)
wi_install_import(TARGETS message::msg)
wi_install_dll_bin(DIRS ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
wi_install_dll_dir(DIRS ${CMAKE_BINARY_DIR})

cfep_install(msgTargets)
```



## 设置编译选项

您可以使用 `set` 命令来设置编译选项，例如：

```cmake
set(CMAKE_CXX_FLAGS "-Wall -Wextra")
```

这将向 C++ 编译器传递 `-Wall` 和 `-Wextra` 选项，以启用更多的编译警告。

## 设置构建类型

您可以使用 `CMAKE_BUILD_TYPE` 变量来设置构建类型。构建类型指定了编译器应该如何优化代码。常用的构建类型有：

- Debug：用于调试，不进行优化，生成调试符号。
- Release：用于发布，进行最大程度的优化。
- RelWithDebInfo：进行优化，并生成调试符号。

例如：

```cmake
set(CMAKE_BUILD_TYPE Debug)
```

## 添加源文件

您可以使用 `add_executable` 命令来添加源文件，例如：

```cmake
add_executable(myapp main.cpp foo.cpp bar.cpp)
```

这将添加 main.cpp、foo.cpp 和 bar.cpp 三个源文件。

## 添加库文件

您可以使用 `target_link_libraries` 命令来链接库文件，例如：

```cmake
target_link_libraries(myapp pthread)
```

这将链接 pthread 库文件。

## 添加子目录

如果您的项目包含多个子目录，您可以使用 `add_subdirectory` 命令来添加子目录，例如：

```cmake
add_subdirectory(src)
```

这将添加名为 src 的子目录，并在其中查找 CMakeLists.txt 文件。

## 安装目标文件

您可以使用 `install` 命令来安装目标文件，例如：

```cmake
install(TARGETS myapp DESTINATION bin)
```

这将安装 myapp 可执行文件到 bin 目录中。

## 自定义构建过程

您可以使用 `add_custom_command` 命令来添加自定义构建过程，例如：

```cmake
rubyadd_custom_command(
    TARGET myapp POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:myapp> ${CMAKE_BINARY_DIR}/bin
)
```

这将在构建完 myapp 可执行文件后执行一个命令，将 myapp 复制到 bin 目录中。

## CMake 模块

CMake 模块是一组 CMake 指令，用于简化 CMakeLists.txt 文件的编写。常用的 CMake 模块有：

- `FindXXX.cmake`：用于查找库文件。
- `CheckXXX.cmake`：用于检查系统特性，例如检查是否存在某个函数或某个头文件。
- `MacroXXX.cmake`：用于定义宏。
- `FunctionXXX.cmake`：用于定义函数。

您可以使用 `include` 命令来包含 CMake 模块，例如：

```cmake
include(FindOpenSSL)
```

这将包含 FindOpenSSL.cmake 模块。

## 使用变量

您可以使用变量来简化 CMakeLists.txt 文件的编写。变量可以保存字符串、列表、布尔值等类型的值。以下是一些常用的变量：

- `PROJECT_NAME`：项目名称。
- `CMAKE_SOURCE_DIR`：项目根目录。
- `CMAKE_BINARY_DIR`：构建目录。
- `CMAKE_CXX_COMPILER`：C++ 编译器。
- `CMAKE_CXX_FLAGS`：C++ 编译选项。
- `CMAKE_INSTALL_PREFIX`：安装目录。
- `CMAKE_BUILD_TYPE`：构建类型。

您可以使用 `set` 命令来定义变量，例如：

```cmake
set(MYVAR "hello world")
```

这将定义名为 MYVAR 的变量，并将其值设置为 "hello world"。

您可以使用 `${}` 语法来引用变量，例如：

```cmake
message(STATUS "My var: ${MYVAR}")
```

这将输出 "My var: hello world"。

## 使用条件语句

您可以使用条件语句来根据不同的条件执行不同的操作。常用的条件语句有：

- `if`：如果满足条件，则执行一组操作。
- `else`：如果不满足条件，则执行一组操作。
- `elseif`：如果不满足上一个条件，但满足当前条件，则执行一组操作。
- `endif`：结束条件语句块。

例如：

```cmake
if(MYVAR STREQUAL "hello")
    message(STATUS "My var is hello")
elseif(MYVAR STREQUAL "world")
    message(STATUS "My var is world")
else()
    message(STATUS "My var is neither hello nor world")
endif()
```

这将根据 MYVAR 变量的值输出不同的消息。

## 使用循环语句

您可以使用循环语句来重复执行某些操作。常用的循环语句有：

- `foreach`：遍历列表。
- `while`：当满足条件时，重复执行一组操作。
- `break`：退出循环。
- `continue`：跳过本次循环。

例如：

```cmake
foreach(FILENAME IN ITEMS foo.cpp bar.cpp baz.cpp)
    message(STATUS "Processing file: ${FILENAME}")
endforeach()
```

这将遍历名为 FILENAME 的变量，并输出每个文件的名称。

## 使用函数和宏

您可以使用函数和宏来简化 CMakeLists.txt 文件的编写。函数和宏可以接受参数，并返回值。以下是一个使用函数的例子：

```cmake
phpfunction(my_function ARG1 ARG2)
    message(STATUS "Argument 1: ${ARG1}")
    message(STATUS "Argument 2: ${ARG2}")
endfunction()

my_function("hello" "world")
```

这将定义名为 my_function 的函数，并调用该函数。

以下是一个使用宏的例子：

```cmake
macro(my_macro ARG1 ARG2)
    message(STATUS "Argument 1: ${ARG1}")
    message(STATUS "Argument 2: ${ARG2}")
endmacro()

my_macro("hello" "world")
```

这将定义名为 my_macro 的宏，并调用该宏。

## CMake 的工具链文件

CMake 的工具链文件（toolchain file）是一个 CMake 脚本，用于配置交叉编译工具链。工具链文件通常包含以下内容：

- 设置交叉编译工具链路径。
- 定义交叉编译器和链接器。
- 设置交叉编译选项。
- 定义交叉编译库路径。
- 设置交句柄交叉编译系统头文件路径。
- 定义其他交叉编译选项。

您可以使用 `-DCMAKE_TOOLCHAIN_FILE` 参数将工具链文件传递给 CMake。

例如，假设您正在交叉编译一个 ARM 平台的项目，可以创建以下工具链文件：

```cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(CMAKE_C_COMPILER arm-linux-gcc)
set(CMAKE_CXX_COMPILER arm-linux-g++)
set(CMAKE_STRIP arm-linux-strip)

set(CMAKE_FIND_ROOT_PATH /usr/arm-linux-gnueabihf)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

## CMake 的插件机制

CMake 的插件机制允许您编写自定义插件，以扩展 CMake 的功能。插件通常包括以下组件：

- 模块文件（module file）：定义 CMake 变量、函数和宏的脚本文件。
- 脚本文件（script file）：包含要执行的 CMake 命令的脚本文件。
- 配置文件（config file）：包含插件的元数据和依赖项信息的文件。
- 安装文件（install file）：指定插件的安装路径和其他相关信息的文件。

CMake 的插件机制使用 `find_package` 命令来查找和加载插件。`find_package` 命令搜索 `CMAKE_PREFIX_PATH` 环境变量指定的目录，以查找与指定名称匹配的配置文件。如果找到了配置文件，则加载插件。

例如，假设您正在使用一个名为 `MyPlugin` 的插件，可以创建以下配置文件：

```cmake
lessset(MyPlugin_VERSION 1.0)
set(MyPlugin_INCLUDE_DIRS /usr/local/include)
set(MyPlugin_LIBRARIES myplugin)

configure_file(MyPluginConfig.cmake.in MyPluginConfig.cmake @ONLY)
```

此文件定义了插件的元数据和依赖项信息，并使用 `configure_file` 命令生成配置文件。`MyPluginConfig.cmake.in` 文件包含以下内容：

```cmake
set(MyPlugin_INCLUDE_DIRS "@MyPlugin_INCLUDE_DIRS@")
set(MyPlugin_LIBRARIES "@MyPlugin_LIBRARIES@")
```

此文件定义了 `MyPlugin_INCLUDE_DIRS` 和 `MyPlugin_LIBRARIES` 变量，并使用 `@` 符号作为变量引用的占位符。

然后，在您的项目中，您可以使用以下命令来查找和加载 `MyPlugin` 插件：

```cmake
find_package(MyPlugin 1.0 REQUIRED)
include_directories(${MyPlugin_INCLUDE_DIRS})
```

## CMake 的测试

CMake 支持多种类型的测试，包括单元测试和集成测试。

### 单元测试

CMake 的 `add_test` 命令用于添加单元测试。该命令接受以下参数：

- `NAME`：测试的名称。
- `COMMAND`：测试命令。
- `WORKING_DIRECTORY`：测试工作目录。
- `ENVIRONMENT`：测试环境变量。

例如，假设您正在测试一个名为 `MyLibrary` 的库，可以创建以下测试：

```cmake
add_executable(MyLibraryTest MyLibraryTest.cpp)
target_link_libraries(MyLibraryTest MyLibrary)

add_test(NAME MyLibraryTest COMMAND MyLibraryTest)
```

此代码将创建一个名为 `MyLibraryTest` 的可执行文件，并将 `MyLibrary` 库链接到它。然后，使用 `add_test` 命令添加一个名为 `MyLibraryTest` 的测试，并将 `MyLibraryTest` 可执行文件作为测试命令。

### 集成测试

CMake 的 `add_custom_target` 和 `add_custom_command` 命令用于创建集成测试。集成测试是一种测试整个系统的方法，它通常涉及多个组件和模块。

`add_custom_target` 命令创建一个自定义目标，该目标不生成任何文件，但可以作为依赖项添加到其他目标中。

`add_custom_command` 命令创建一个自定义命令，并将其添加到自定义目标中。自定义命令通常是一个脚本文件，该文件执行集成测试并将结果写入日志文件。

例如，假设您正在测试一个名为 `MySystem` 的系统，可以创建以下集成测试：

```cmake
add_custom_target(MySystemTest
    COMMAND ${CMAKE_COMMAND} -E echo "Running MySystemTest"
    COMMAND ${CMAKE_COMMAND} -E touch ${CMAKE_BINARY_DIR}/MySystemTest.log
    DEPENDS MySystem
)

add_custom_command(TARGET MySystemTest POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E echo "MySystemTest results:"
    COMMAND ${CMAKE_COMMAND} -E cat ${CMAKE_BINARY_DIR}/MySystemTest.log
)
```

此代码将创建一个名为 `MySystemTest` 的自定义目标，并将其添加到构建中。`MySystemTest` 自定义目标包括两个自定义命令，第一个命令运行测试，并将结果写入日志文件。第二个命令在测试完成后运行，并输出日志文件中的测试结果。

## CMake 的其他功能

除了上述功能外，CMake 还提供了许多其他有用的功能，例如：

- `add_dependencies` 命令：用于定义目标之间的依赖关系。
- `install` 命令：用于将构建的文件和目录安装到系统中。
- `set_target_properties` 命令：用于设置目标的属性，例如输出名称、编译标志和链接标志。
- `cmake_policy` 命令：用于设置 CMake 的策略版本。
- `message` 命令：用于输出消息到 CMake 的日志中。

例如，以下代码演示了如何使用这些命令：

```cmake
add_library(MyLibrary MyLibrary.cpp)

add_executable(MyExecutable MyExecutable.cpp)
target_link_libraries(MyExecutable MyLibrary)

add_test(NAME MyExecutableTest COMMAND MyExecutable)

install(TARGETS MyLibrary MyExecutable
    DESTINATION bin
)

set_target_properties(MyExecutable
    PROPERTIES
    OUTPUT_NAME "myexe"
)

add_dependencies(MyExecutable MyLibrary)

cmake_policy(VERSION 3.13)

message("This is a message")
```

此代码定义了一个名为 `MyLibrary` 的库和一个名为 `MyExecutable` 的可执行文件，后者链接到前者。然后，它使用 `add_test` 命令定义了一个名为 `MyExecutableTest` 的测试。

接下来，`install` 命令将 `MyLibrary` 和 `MyExecutable` 安装到 `bin` 目录中。`set_target_properties` 命令将 `MyExecutable` 的输出名称设置为 `myexe`。

`add_dependencies` 命令定义了 `MyExecutable` 依赖于 `MyLibrary`。`cmake_policy` 命令设置 CMake 的策略版本为 3.13。最后，`message` 命令输出了一个消息到 CMake 的日志中。

## 总结

CMake 是一个功能强大的构建系统，可帮助您管理 C++ 项目的构建过程。本文提供了一个概述，介绍了 CMake 的基本概念、用法和功能。希望这些信息能够帮助您入门 CMake，并更好地管理您的 C++ 项目。