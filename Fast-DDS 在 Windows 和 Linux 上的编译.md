# Fast-DDS 在 Windows 和 Linux 上的编译

经测试，编译构建步骤适用于`3.3.0`，并在`Windows 11 23H2`和`Kylin V10 SP1 x86_64 2503`上编译通过。

## 源码准备

```bash
git clone -b v3.3.0 https://github.com/eProsima/Fast-DDS.git
mv Fast-DDS Fast-DDS-3.3.0
cd Fast-DDS-3.3.0
git submodule update --init
git submodule add https://github.com/foonathan/memory.git thirdparty/foonathan-memory
```

修改根目录下的 `CMakeLists.txt`，在靠前位置添加：

```cmake
add_compile_options($<$<C_COMPILER_ID:MSVC>:/utf-8>)
```

用以消除以下警告：

> warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失

## Windows 上的编译

### 编译 foonathan-memory

编译脚本（批处理文件）如下：

```bat
@echo off

FOR /F "tokens=* usebackq" %%i IN (`vswhere.exe -latest -property installationPath`) DO (
	set VCINSTALLATIONPATH=%%i
	call "%%i\VC\Auxiliary\Build\vcvars64.bat"
	set VC_REDIST="%%i\VC\Redist\MSVC\v143\vc_redist.x64.exe"
)

set ROOT=%CD%

if exist build rmdir /S /Q build
md build
cd build

cmake -G "Ninja" -DBUILD_SHARED_LIBS=OFF -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DFOONATHAN_MEMORY_BUILD_EXAMPLES=OFF -DFOONATHAN_MEMORY_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=install ..
IF %ERRORLEVEL% NEQ 0 GOTO ERROR

ninja
IF %ERRORLEVEL% NEQ 0 GOTO ERROR
echo build succeed!

ninja install
IF %ERRORLEVEL% NEQ 0 GOTO ERROR
echo install succeed!

pause
exit /B 0

:ERROR
echo build failed!
pause
exit /B 1
```

### 编译 Fast-DDS

`CMakeSettings.json` 内容如下：

```json
{
  "configurations": [
    {
      "name": "x64-Debug",
      "generator": "Ninja",
      "configurationType": "Debug",
      "inheritEnvironments": [ "msvc_x64_x64" ],
      "buildRoot": "${projectDir}\\out\\build\\${name}",
      "installRoot": "${projectDir}\\out\\install\\${name}",
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": "",
      "variables": [
        {
          "name": "THIRDPARTY",
          "value": "ON",
          "type": "STRING"
        },
        {
          "name": "THIRDPARTY_Asio",
          "value": "ON",
          "type": "STRING"
        },
        {
          "name": "THIRDPARTY_fastcdr",
          "value": "ON",
          "type": "STRING"
        },
        {
          "name": "NO_TLS",
          "value": "True",
          "type": "BOOL"
        },
        {
          "name": "foonathan_memory_DIR",
          "value": "D:/Code/Projects/Fast-DDS-3.3.0/thirdparty/foonathan-memory/install/share/foonathan_memory/cmake",
          "type": "PATH"
        },
        {
          "name": "COMPILE_EXAMPLES",
          "value": "True",
          "type": "BOOL"
        },
        {
          "name": "BUILD_SHARED_LIBS",
          "value": "False",
          "type": "BOOL"
        },
        {
          "name": "INSTALL_EXAMPLES",
          "value": "True",
          "type": "BOOL"
        }
      ]
    },
    {
      "name": "x64-Release",
      "generator": "Ninja",
      "configurationType": "RelWithDebInfo",
      "buildRoot": "${projectDir}\\out\\build\\${name}",
      "installRoot": "${projectDir}\\out\\install\\${name}",
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": "",
      "inheritEnvironments": [ "msvc_x64_x64" ],
      "variables": [
        {
          "name": "THIRDPARTY",
          "value": "ON",
          "type": "STRING"
        },
        {
          "name": "THIRDPARTY_Asio",
          "value": "ON",
          "type": "STRING"
        },
        {
          "name": "THIRDPARTY_fastcdr",
          "value": "ON",
          "type": "STRING"
        },
        {
          "name": "NO_TLS",
          "value": "True",
          "type": "BOOL"
        },
        {
          "name": "foonathan_memory_DIR",
          "value": "D:/Code/Projects/Fast-DDS-3.3.0/thirdparty/foonathan-memory/install/share/foonathan_memory/cmake",
          "type": "PATH"
        },
        {
          "name": "COMPILE_EXAMPLES",
          "value": "True",
          "type": "BOOL"
        },
        {
          "name": "BUILD_SHARED_LIBS",
          "value": "False",
          "type": "BOOL"
        },
        {
          "name": "INSTALL_EXAMPLES",
          "value": "True",
          "type": "BOOL"
        }
      ]
    }
  ]
}
```

**修改上述"foonathan_memory_DIR"的路径**，然后使用 Visual Studio，正常编译并安装即可（推荐使用 VS 2019）。

注意上述配置设置了`NO_TLS`，从而不再依赖 `OpenSSH`，可按需启用。

## Linux 上的编译

编译脚本如下：

```bash
#!/bin/bash

# 启用错误检查（任何命令失败立即退出）
set -e

cd "thirdparty/foonathan-memory" || exit 1
if [[ -d "install" ]]; then
    echo "foonathan-memory 已编译安装"
else
    echo "编译 foonathan-memory 库..."
    cmake -B "build" -G "Ninja" \
        -DBUILD_SHARED_LIBS=OFF \
        -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
        -DFOONATHAN_MEMORY_BUILD_EXAMPLES=OFF \
        -DFOONATHAN_MEMORY_BUILD_TESTS=OFF \
        -DCMAKE_INSTALL_PREFIX="install"
fi

cmake --build "build" -j

cmake --install "build"

cd ../..

echo "编译主项目..."
cmake -B "build" -G "Ninja" \
    -DCMAKE_INSTALL_PREFIX="install" \
    -DBUILD_SHARED_LIBS=OFF \
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
    -DTHIRDPARTY=ON \
    -DTHIRDPARTY_Asio=ON \
    -DTHIRDPARTY_fastcdr=ON \
    -DNO_TLS=ON \
    -DCOMPILE_EXAMPLES=ON \
    -DINSTALL_EXAMPLES=ON \
    -DSANITIZER=Thread \
    -Dfoonathan_memory_DIR="thirdparty/foonathan-memory/install/lib/foonathan_memory/cmake" \
    -DCMAKE_BUILD_TYPE=Release

cmake --build "build" -j --verbose

cmake --install "build"

echo "编译安装完成！"
```

## 番外：Fast DDS-Gen 的使用

Fast DDS-Gen 是一个 Java 应用程序，它使用 IDL（接口定义语言）文件中定义的数据类型生成 C++ 源代码。我们跳过编译，直接使用官方提供的预编译好的 Fast DDS-Gen 工具。

使用 Fast DDS-Gen，需提前准备 Java 环境，推荐 [Temurin JDK](https://adoptium.net/zh-CN/temurin/releases)，它是由 Eclipse Adoptium 项目维护的免费开源 OpenJDK 发行版，提供与 Java SE 标准兼容的高性能 Java 开发与运行时环境。

接着从[eProsima 官网](https://www.eprosima.com/product-download)下载 Fast-DDS 安装包（例如：eProsima_Fast-DDS-3.2.2-Windows.exe），可仅安装 Fast DDS-Gen。

打开 Developer Command Prompt for VS 2019，切换到 IDL 文件目录（这里是 `D:\Example`）下，运行：

```cmd
D:\Example>"C:\Program Files\eProsima\fastdds 3.2.2\bin\fastddsgen.bat" Data.idl
```

生成的 c++ 代码文件在这个 IDL 相关的后续开发中需要使用。
