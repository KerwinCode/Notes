# OpenDDS 在 Windows 和 Linux 上的编译

经测试，编译构建步骤适用于`3.33.0`和 `3.31.0`，并在`Windows 11 23H2`和`Kylin V10 SP1 x86_64 2503`上编译通过。

## 离线编译准备

如果不需要离线构建，请在下载 OpenDDS 源码以及安装 Perl 后跳过本节。

下载以下文件：

1. [OpenDDS 源码](https://github.com/OpenDDS/OpenDDS/releases)（以 3.33.0 版本为例，**同时注意 CMake 版本不低于 4.0**）

2. [Perl](https://www.perl.org/get.html)（Windows 上使用 [Strawberry Perl](http://strawberryperl.com/)，Linux 上可以使用包管理器安装或通过源码编译安装）

3. [rapidjson-fd3dc29a5c2852df569e1ea81dbde2c412ac5051.zip](https://github.com/Tencent/rapidjson/archive/fd3dc29a5c2852df569e1ea81dbde2c412ac5051.zip)

4. [ACE+TAO-src-7.1.3.tar.bz2](https://github.com/DOCGroup/ACE_TAO/releases/download/ACE%2BTAO-7_1_3/ACE+TAO-7.1.3.tar.bz2)

5. [ACE+TAO-src-7.1.3.zip](https://github.com/DOCGroup/ACE_TAO/releases/download/ACE%2BTAO-7_1_3/ACE+TAO-7.1.3.zip)

6. [xerces-c-3.3.0.zip](https://dlcdn.apache.org//xerces/c/3/sources/xerces-c-3.3.0.zip)（可选，用于以 XML 文件设置 QoS 时使用）

2 - 5 项也可以使用 CMake 自动拉取，`rapidjson-src` 在`build/_deps/rapidjson-src`，`ACE+TAO-src-7.1.3` 的这两个文件位置在`build/_deps/ace_tao_dl-tmp`，可全部拷贝到源码 deps 目录下。

同时还需要做以下编译前准备：

1. 注释 `.gitmodules` 所有内容

2. `rapidjson`依赖处理

    修改源码根目录下的`CMakeLists.txt`，50 行左右：

    ```cmake
    if(NOT DEFINED OPENDDS_RAPIDJSON)
      set(OPENDDS_RAPIDJSON "${OPENDDS_SOURCE_DIR}/tools/rapidjson")
      if(NOT EXISTS "${OPENDDS_RAPIDJSON}/include")
        FetchContent_Declare(rapidjson
          # GIT_REPOSITORY https://github.com/Tencent/rapidjson.git
          # GIT_TAG fd3dc29a5c2852df569e1ea81dbde2c412ac5051
          SOURCE_DIR "${CMAKE_SOURCE_DIR}/deps/rapidjson-src"
          # 或者改为，但注意要对应上述 commit id 下载对应版本：
          # URL "${CMAKE_SOURCE_DIR}/rapidjson-fd3dc29a5c2852df569e1ea81dbde2c412ac5051.zip"
        )
        FetchContent_Populate(rapidjson)
        set(OPENDDS_RAPIDJSON "${rapidjson_SOURCE_DIR}")
      endif()
    endif()
    ```

3. `ACE+TAO`依赖处理

    修改源码根目录下的`acetao.ini`，30 行 和 36 行，压缩包 URL 改为本地路径（可填写例如`${CMAKE_SOURCE_DIR}/deps/ACE+TAO-src-7.1.3.zip`的形式，需要 CMake 4.0 以上版本的支持）

4. `Xerces-C`依赖处理（可选）

    解压 `xerces-c-3.3.0.zip`，并编译安装（可编译为静态库：设置 CMake 变量 `"BUILD_SHARED_LIBS": "OFF"`）

    并设置 OpenDDS 项目的 CMake 变量：`"OPENDDS_XERCES3": "${sourceDir}/deps/xerces-c-3.3.0/install/${presetName}"`

    可写入`CMakePresets.json`文件中。

    Linux 上编译`Xerces-C`，需要加上`-fPIC`，可设置 CMake 变量 `"CMAKE_POSITION_INDEPENDENT_CODE": "ON"`。

    若需要静态链接`Xerces-C`，还需修改`cmake\init.cmake`，在 659 行左右添加：
    
    ```cmake
    # This should be in ace_group.cmake, but it's needed by build_ace_tao.cmake.
    if(OPENDDS_XERCES3)
      # 添加下面一行：
      find_package(Threads REQUIRED)
      find_package(XercesC PATHS "${OPENDDS_XERCES3}" NO_DEFAULT_PATH QUIET)
    ```

## Linux 上的编译

`ACE+TAO`编译时添加`-fPIC`

`cmake/build_ace_tao.cmake` 92 行左右添加：

```cmake
  if(UNIX)
    list(APPEND _build_cmd
      "opendds_cmake_pic=-fPIC"
    )
  endif()
```

`cmake/configure_ace_tao.pl` 113 行左右添加：

```perl
print $platform_macros_file ("CCFLAGS += \$(opendds_cmake_pic)\n");
```

根目录下的`CMakeLisis.txt`第 2 行左右添加：

```cmake
if(UNIX)
  set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
endif()
```

然后使用 CMake，正常编译安装即可。

## Windows 上的编译

Windows 上使用 VS 2022 编译需要注意：

1. "generator" 只能选择 "Visual Studio 17 2022 Win64"，否则会生成错误的 sln。
2. "configurationType" 只能使用 "Debug" 或 "Release"，不能使用 "RelWithDebInfo"。

然后正常编译安装即可。

## 使用 XML_QoS

依赖 Xerces-C 库，需要指定`OPENDDS_XERCES3`变量，指向 Xerces-C 的安装目录，例如：

```cmake
-DOPENDDS_XERCES3=/path/to/xerces-c-3.3.0/install/linux-debug
```

## 参考 CMakePresets.json

可使用以下 CMake 预设，该预设支持 Windows / Linux 跨平台编译：

```json
{
  "version": 3,
  "cmakeMinimumRequired": {
    "major": 3,
    "minor": 21,
    "patch": 0
  },
  "configurePresets": [
    {
      "name": "base-all",
      "hidden": true,
      "cacheVariables": {
        "CMAKE_EXPORT_COMPILE_COMMANDS": "ON",
        "CMAKE_CXX_STANDARD": "17",
        "CMAKE_CXX_STANDARD_REQUIRED": "ON"
      }
    },
    {
      "name": "base-windows",
      "hidden": true,
      "inherits": "base-all",
      "description": "Base settings for Windows (MSVC)",
      "generator": "Visual Studio 17 2022",
      "architecture": "x64",
      "toolset": "host=x64"
    },
    {
      "name": "base-linux",
      "hidden": true,
      "inherits": "base-all",
      "description": "Base settings for Linux (GCC/Clang with Ninja)",
      "generator": "Ninja"
    },
    {
      "name": "base-windows-debug",
      "hidden": true,
      "inherits": "base-windows",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "Debug" },
      "environment": {
        "CXXFLAGS": "/DWIN32 /D_WINDOWS /W4 /permissive- /EHsc /Od /Zi",
        "CFLAGS": "/DWIN32 /D_WINDOWS /W4 /Od /Zi"
      }
    },
    {
      "name": "base-windows-release",
      "hidden": true,
      "inherits": "base-windows",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "Release" },
      "environment": {
        "CXXFLAGS": "/DWIN32 /D_WINDOWS /W4 /permissive- /EHsc /O2 /Ob2 /DNDEBUG",
        "CFLAGS": "/DWIN32 /D_WINDOWS /W4 /O2 /Ob2 /DNDEBUG"
      }
    },
    {
      "name": "base-windows-relwithdebinfo",
      "hidden": true,
      "inherits": "base-windows",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "RelWithDebInfo" },
      "environment": {
        "CXXFLAGS": "/DWIN32 /D_WINDOWS /W4 /permissive- /EHsc /O2 /Ob2 /DNDEBUG /Zi",
        "CFLAGS": "/DWIN32 /D_WINDOWS /W4 /O2 /Ob2 /DNDEBUG /Zi"
      }
    },
    {
      "name": "base-linux-debug",
      "hidden": true,
      "inherits": "base-linux",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "Debug" },
      "environment": {
        "CXXFLAGS": "-Wall -Wextra -Wpedantic -g",
        "CFLAGS": "-Wall -Wextra -Wpedantic -g"
      }
    },
    {
      "name": "base-linux-release",
      "hidden": true,
      "inherits": "base-linux",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "Release" },
      "environment": {
        "CXXFLAGS": "-Wall -Wextra -Wpedantic -O3 -DNDEBUG",
        "CFLAGS": "-Wall -Wextra -Wpedantic -O3 -DNDEBUG"
      }
    },
    {
      "name": "base-linux-relwithdebinfo",
      "hidden": true,
      "inherits": "base-linux",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "RelWithDebInfo" },
      "environment": {
        "CXXFLAGS": "-Wall -Wextra -Wpedantic -O2 -g -DNDEBUG",
        "CFLAGS": "-Wall -Wextra -Wpedantic -O2 -g -DNDEBUG"
      }
    },
    {
      "name": "windows-debug",
      "displayName": "Debug (Windows)",
      "inherits": "base-windows-debug",
      "binaryDir": "${sourceDir}/build/${presetName}",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/install/${presetName}",
        "OPENDDS_XERCES3": "${sourceDir}/deps/xerces-c-3.3.0/install/${presetName}"
      }
    },
    {
      "name": "windows-release",
      "displayName": "Release (Windows)",
      "inherits": "base-windows-release",
      "binaryDir": "${sourceDir}/build/${presetName}",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/install/${presetName}",
        "OPENDDS_XERCES3": "${sourceDir}/deps/xerces-c-3.3.0/install/${presetName}"
      }
    },
    {
      "name": "windows-relwithdebinfo",
      "displayName": "RelWithDebInfo (Windows)",
      "inherits": "base-windows-relwithdebinfo",
      "binaryDir": "${sourceDir}/build/${presetName}",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/install/${presetName}",
        "OPENDDS_XERCES3": "${sourceDir}/deps/xerces-c-3.3.0/install/${presetName}"
      }
    },
    {
      "name": "linux-debug",
      "displayName": "Debug (Linux)",
      "inherits": "base-linux-debug",
      "binaryDir": "${sourceDir}/build/${presetName}",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/install/${presetName}",
        "OPENDDS_XERCES3": "${sourceDir}/deps/xerces-c-3.3.0/install/${presetName}"
      }
    },
    {
      "name": "linux-release",
      "displayName": "Release (Linux)",
      "inherits": "base-linux-release",
      "binaryDir": "${sourceDir}/build/${presetName}",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/install/${presetName}",
        "OPENDDS_XERCES3": "${sourceDir}/deps/xerces-c-3.3.0/install/${presetName}"
      }
    },
    {
      "name": "linux-relwithdebinfo",
      "displayName": "RelWithDebInfo (Linux)",
      "inherits": "base-linux-relwithdebinfo",
      "binaryDir": "${sourceDir}/build/${presetName}",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/install/${presetName}",
        "OPENDDS_XERCES3": "${sourceDir}/deps/xerces-c-3.3.0/install/${presetName}"
      }
    }
  ],
  "buildPresets": [
    {
      "name": "windows-debug",
      "configurePreset": "windows-debug",
      "configuration": "Debug"
    },
    {
      "name": "windows-release",
      "configurePreset": "windows-release",
      "configuration": "Release"
    },
    {
      "name": "windows-relwithdebinfo",
      "configurePreset": "windows-relwithdebinfo",
      "configuration": "RelWithDebInfo"
    },
    {
      "name": "linux-debug",
      "configurePreset": "linux-debug",
      "configuration": "Debug"
    },
    {
      "name": "linux-release",
      "configurePreset": "linux-release",
      "configuration": "Release"
    },
    {
      "name": "linux-relwithdebinfo",
      "configurePreset": "linux-relwithdebinfo",
      "configuration": "RelWithDebInfo"
    }
  ],
  "testPresets": [
    {
      "name": "windows-debug",
      "configurePreset": "windows-debug",
      "configuration": "Debug",
      "output": { "outputOnFailure": true }
    },
    {
      "name": "windows-release",
      "configurePreset": "windows-release",
      "configuration": "Release",
      "output": { "outputOnFailure": true }
    },
    {
      "name": "windows-relwithdebinfo",
      "configurePreset": "windows-relwithdebinfo",
      "configuration": "RelWithDebInfo",
      "output": { "outputOnFailure": true }
    },
    {
      "name": "linux-debug",
      "configurePreset": "linux-debug",
      "configuration": "Debug",
      "output": { "outputOnFailure": true }
    },
    {
      "name": "linux-release",
      "configurePreset": "linux-release",
      "configuration": "Release",
      "output": { "outputOnFailure": true }
    },
    {
      "name": "linux-relwithdebinfo",
      "configurePreset": "linux-relwithdebinfo",
      "configuration": "RelWithDebInfo",
      "output": { "outputOnFailure": true }
    }
  ]
}
```