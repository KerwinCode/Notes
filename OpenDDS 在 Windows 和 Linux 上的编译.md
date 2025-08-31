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

后三项也可以使用 CMake 自动拉取，`rapidjson-src` 在`build/_deps/rapidjson-src`，`ACE+TAO-src-7.1.3` 的这两个文件位置在`build/_deps/ace_tao_dl-tmp`，可全部拷贝到源码目录下。

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
          SOURCE_DIR "${CMAKE_SOURCE_DIR}/rapidjson-src"
          # 或者改为，但注意要对应上述 commit id 下载对应版本：
          # URL "${CMAKE_SOURCE_DIR}/rapidjson-fd3dc29a5c2852df569e1ea81dbde2c412ac5051.zip"
        )
        FetchContent_Populate(rapidjson)
        set(OPENDDS_RAPIDJSON "${rapidjson_SOURCE_DIR}")
      endif()
    endif()
    ```

3. `ACE+TAO`依赖处理

    修改源码根目录下的`acetao.ini`，30 行 和 36 行，压缩包 URL 改为本地路径（可填写例如`${CMAKE_SOURCE_DIR}/deps/ACE+TAO-src-7.1.3.zip`的形式）

## Linux 上的编译

`ACE+TAO`编译时添加`-fPIC`

`cmake/build_ace_tao.pl` 90 行左右添加：

```cmake
  if(UNIX)
    list(APPEND _build_cmd
      "opendds_cmake_pic=-fPIC"
    )
  endif()
```

`cmake/configure_ace_tao.pl` 112 行左右添加：

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

Windows 上使用 VS 2022 编译：

`CMakeSettings.json` 内容如下：

```json
{
  "configurations": [
    {
      "name": "x64-Debug",
      "generator": "Visual Studio 17 2022 Win64",
      "configurationType": "Debug",
      "inheritEnvironments": [ "msvc_x64_x64" ],
      "buildRoot": "${projectDir}\\out\\build\\${name}",
      "installRoot": "${projectDir}\\out\\install\\${name}",
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": ""
    },
    {
      "name": "x64-Release",
      "generator": "Visual Studio 17 2022 Win64",
      "configurationType": "Release",
      "buildRoot": "${projectDir}\\out\\build\\${name}",
      "installRoot": "${projectDir}\\out\\install\\${name}",
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": "",
      "inheritEnvironments": [ "msvc_x64_x64" ],
      "variables": []
    }
  ]
}
```

需要注意的是：

1. "generator" 只能选择 "Visual Studio 17 2022 Win64"，否则会生成错误的 sln。
2. "configurationType" 只能使用 "Debug" 或 "Release"，不能使用 "RelWithDebInfo"。

然后正常编译安装即可。
