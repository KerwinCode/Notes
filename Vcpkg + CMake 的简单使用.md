# Vcpkg + CMake 的简单使用

## 构建 vcpkg

```bash
# 克隆 vcpkg 仓库
git clone https://github.com/microsoft/vcpkg.git

# 进入 vcpkg 目录
cd vcpkg

# 在 Linux/macOS 上运行引导脚本
./bootstrap-vcpkg.sh

# 在 Windows 上运行引导脚本
.\bootstrap-vcpkg.bat
```

成功后可以在目录下看到 vcpkg.exe

## 配置环境变量

系统变量中添加 VCPKG_ROOT 值为 刚才构建的目录

并将 %VCPKG_ROOT% 添加到 Path 中

## 集成 vcpkg 到 CMake

```bat
# 运行
vcpkg integrate install
# 可看到以下提示
Applied user-wide integration for this vcpkg root.
CMake projects should use: "-DCMAKE_TOOLCHAIN_FILE=C:/Apps/vcpkg/scripts/buildsystems/vcpkg.cmake"

All MSBuild C++ projects can now #include any installed libraries. Linking will be handled automatically. Installing new libraries will make them instantly available.
```

在 VS 2022 中新建 CMake 工程，并在工程目录下运行 PowerShell 命令行：

```bat
# 进入项目目录
cd ...

# 步骤1: 创建应用清单
vcpkg new --application
# 为应用程序初始化项目依赖管理，生成以下两个核心文件：
# vcpkg.json 声明项目为应用程序（非库），无需指定名称和版本，简化配置。
# vcpkg-configuration.json 定义依赖库的下载源（官方仓库 + 实验性预构建库）

# 步骤2: 添加依赖项
vcpkg add port spdlog
# port 是 vcpkg 包管理器的核心概念，用于定义第三方库的构建、安装规则

# 步骤3: 安装所有依赖（可省略）
vcpkg install
```

设置 CMAKE_TOOLCHAIN_FILE ：

```bat
# 方法1：命令行传递​：
cmake -B build -DCMAKE_TOOLCHAIN_FILE=C:/Apps/vcpkg/scripts/buildsystems/vcpkg.cmake

# 方法2：在 CMakeLists.txt 中设置（需在 project() 前）​​：
set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")

# 方法3：修改 CMakePresets.json, 在 cacheVariables 中添加（实测使用全局安装的库时失效）：
"CMAKE_TOOLCHAIN_FILE": "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"

```

修改 CMakeLists.txt ：

```cmake
# 追加：
find_package(spdlog CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE spdlog::spdlog)
```

添加示例代码：

```cpp
#include <spdlog/spdlog.h>

int main()
{
    spdlog::info("Welcome to spdlog!");
    spdlog::error("Some error message with arg: {}", 1);

    spdlog::warn("Easy padding in numbers like {:08d}", 12);
    spdlog::critical("Support for int: {0:d};  hex: {0:x};  oct: {0:o}; bin: {0:b}", 42);
    spdlog::info("Support for floats {:03.2f}", 1.23456);
    spdlog::info("Positional args are {1} {0}..", "too", "supported");
    spdlog::info("{:<30}", "left aligned");

    spdlog::set_level(spdlog::level::debug); // Set global log level to debug
    spdlog::debug("This message should be displayed..");

    // change log pattern
    spdlog::set_pattern("[%H:%M:%S %z] [%n] [%^---%L---%$] [thread %t] %v");

    // Compile time log levels
    // Note that this does not change the current log level, it will only
    // remove (depending on SPDLOG_ACTIVE_LEVEL) the call on the release code.
    SPDLOG_TRACE("Some trace message with param {}", 42);
    SPDLOG_DEBUG("Some debug message");
}
```

成功构建并运行。

## 常用命令

```bat
集成到全局：vcpkg integrate install
移除全局：vcpkg integrate remove
集成到工程：vcpkg integrate project（为单个项目生成 NuGet 配置文件（路径：\scripts\buildsystems\*.nupkg），实现项目级依赖隔离）
查看库目录：vcpkg search
查看支持的架构：vcpkg help triplet
指定编译某种架构的程序库：vcpkg install xxxx:x64-windows（x86-windows）
卸载已安装库：vcpkg remove xxxx
卸载库及其所有依赖项：vcpkg remove --purge osg
指定卸载平台：vcpkg remove xxxx:x64-windows
移除所有旧版本库：vcpkg remove --outdated
查看已经安装的库：vcpkg list
更新已经安装的库：vcpkg update xxx
导出已经安装的库：vcpkg export xxxx --7zip（–7zip –raw –nuget –ifw –zip）
```
