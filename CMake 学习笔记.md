# CMake 学习笔记

## 设置 C++ 标准

现代 C++ 项目通常需要指定一个最低的 C++ 标准。

```cmake
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

* `CMAKE_CXX_STANDARD`: 设置 C++ 标准，例如 11, 14, 17, 20。
* `CMAKE_CXX_STANDARD_REQUIRED ON`: 强制要求编译器支持所设定的标准，如果不支持则会报错。
* `CMAKE_CXX_EXTENSIONS OFF`: 关闭编译器特定的扩展，使用纯粹的标准 C++。

## 包含头文件目录

```cmake
# 为所有目标添加头文件目录
include_directories(include)

# (推荐) 为特定目标添加头文件目录
target_include_directories(my_target PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
```

* `include_directories`: 会对当前 `CMakeLists.txt` 及所有后续添加的子目录中的目标都生效。
* `target_include_directories`: **(现代 CMake 更推荐的方式)** 只为指定的目标 `my_target` 添加头文件目录。
    * `PUBLIC`: 表示依赖 `my_target` 的其他目标也会自动包含这个头文件目录。
    * `PRIVATE`: 仅 `my_target` 内部使用。
    * `INTERFACE`: `my_target` 自身不使用，但依赖它的其他目标需要使用。

## 链接库

```cmake
# 链接一个库
target_link_libraries(my_executable PRIVATE my_library)

# 链接多个库
target_link_libraries(my_executable PRIVATE
    Boost::filesystem
    Qt5::Core
)
```

* `target_link_libraries`: 将库链接到目标上。同样有 `PUBLIC`, `PRIVATE`, `INTERFACE` 关键字，其含义与 `target_include_directories` 类似，这对于控制依赖传递非常重要。

## 条件编译

根据不同的平台或编译选项来执行不同的逻辑。

```cmake
if(WIN32)
    # Windows 平台的特定设置
    message(STATUS "正在为 Windows 平台配置")
    target_compile_definitions(my_target PRIVATE -DUNICODE -D_UNICODE)
elseif(UNIX AND NOT APPLE)
    # Linux 平台的特定设置
    message(STATUS "正在为 Linux 平台配置")
    target_link_libraries(my_executable PRIVATE pthread)
endif()
```

* `if()/elseif()/endif()`: CMake 的条件判断语句。
* `WIN32`, `UNIX`, `APPLE` 都是 CMake 预定义的变量，用于判断当前操作系统。

## 使用 `find_package` 查找依赖

对于常见的第三方库 (如 Boost, Qt, OpenCV)，使用 `find_package` 是标准做法。

```cmake
cmake_minimum_required(VERSION 3.1)

project(BoostTest)

add_executable(${PROJECT_NAME} main.cpp)

set(BOOST_ROOT "...")
set(Boost_USE_STATIC_LIBS ON)
set(Boost_NO_SYSTEM_PATHS ON)
set(Boost_NO_WARN_NEW_VERSIONS ON)

find_package(Boost 1.88 REQUIRED COMPONENTS system thread locale)

add_executable(boost_test main.cpp)
target_include_directories(${PROJECT_NAME} PRIVATE ${Boost_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE ${Boost_LIBRARIES})
```

```cmake
cmake_minimum_required(VERSION 3.1)

project(helloworld)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

if(CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

if(NOT Qt5_DIR)
    set(Qt5_DIR "C:/Qt/Qt5.14.2/5.14.2/msvc2017_64/lib/cmake/Qt5" CACHE PATH "Path to Qt5")
endif()

find_package(Qt5 COMPONENTS Widgets REQUIRED)

add_executable(helloworld
    mainwindow.ui
    mainwindow.cpp
    main.cpp
    resources.qrc
)
target_link_libraries(helloworld Qt5::Widgets)
```

* `find_package`: 在系统中查找已经安装的库。
* `REQUIRED`: 如果找不到这个包，CMake 会立即报错并停止配置。
* 现代 CMake 的 `find_package` 通常会提供导入的目标（如 `Qt5::Core`），直接链接这些目标就可以自动完成所有必要的配置，非常方便。

## **Linux 上处理依赖**

这段代码的核心目标是在安装可执行文件时，自动查找其依赖的动态链接库（.so 文件），并将这些库文件一并复制到安装目录的 `lib` 文件夹下。这对于确保程序在没有预装相应依赖的环境中也能正常运行至关重要。

```cmake
# 设置目标文件的 RPATH
set_target_properties(${PROJECT_NAME} PROPERTIES
    INSTALL_RPATH "$ORIGIN/lib"
)

# 在安装阶段执行一段代码
install(CODE [[
    # 获取可执行文件的运行时依赖
    file(GET_RUNTIME_DEPENDENCIES
        EXECUTABLES $<TARGET_FILE:project_name>
        RESOLVED_DEPENDENCIES_VAR DEPS
    )
    # 遍历所有依赖并进行拷贝
    foreach(DEP IN LISTS DEPS)
        message(STATUS "正在复制依赖: ${DEP}")
        file(COPY ${DEP} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/ FOLLOW_SYMLINK_CHAIN)
    endforeach()
]])
```

* `set_target_properties`: 这个命令用来设置目标（比如一个可执行文件或库）的属性。
    * `INSTALL_RPATH "$ORIGIN/lib"`: `RPATH` 是一个嵌入在可执行文件或动态库中的路径，用于告诉系统在哪里查找它所依赖的其他库文件。
        * `$ORIGIN` 是一个特殊的变量，表示可执行文件所在的目录。
        * 所以这行代码的作用是：当程序安装后，运行时会到可执行文件同级目录下的 `lib` 文件夹中去寻找依赖库。

* `install(CODE ...)`: 这个命令允许在安装阶段执行指定的 CMake 代码。 这段代码会在 `make install` (或 `cmake --install .`) 执行时运行。
    * `file(GET_RUNTIME_DEPENDENCIES ...)`: 这是 CMake 3.16 及以上版本提供的强大功能，可以自动分析一个可执行文件在运行时需要哪些动态库。
        * `EXECUTABLES $<TARGET_FILE:project_name>`: 指定要分析依赖的可执行文件。`$<TARGET_FILE:project_name>` 是一个生成器表达式，它会返回 `project_name` 这个目标最终生成的文件路径。
        * `RESOLVED_DEPENDENCIES_VAR DEPS`: 将找到的依赖库的绝对路径存储在一个名为 `DEPS` 的 CMake 变量中。
    * `foreach(DEP IN LISTS DEPS)`: 遍历 `DEPS` 变量中的每一个依赖库路径。
    * `file(COPY ... DESTINATION ...)`: 将找到的依赖库文件复制到指定的安装目录下。
        * `${CMAKE_INSTALL_PREFIX}` 是一个 CMake 内置变量，代表安装的根目录 (例如 `/usr/local`)。
        * `FOLLOW_SYMLINK_CHAIN`: 如果依赖是一个符号链接，这个选项会复制链接指向的实际文件，而不是链接本身。

## **文件操作**

这段代码展示了如何在项目构建完成后，以及在配置阶段进行一些常见的文件和目录操作。

```cmake
# 在项目构建后执行自定义命令
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    # 创建目标目录
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/../bin
    # 复制可执行文件到新目录
    COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:${PROJECT_NAME}> $<TARGET_FILE_DIR:${PROJECT_NAME}>/../bin/${PROJECT_NAME}.exe
    # 复制一个目录
    COMMAND ${CMAKE_COMMAND} -E copy_directory tools $<TARGET_FILE_DIR:${PROJECT_NAME}>/../bin/tools
    COMMENT "正在复制可执行文件到 bin 目录"
)

# 直接在 CMake 配置阶段创建目录
make_directory(${CMAKE_BINARY_DIR}/tools)
# 直接在 CMake 配置阶段复制文件
file(COPY ${CMAKE_SOURCE_DIR}/thirdparty/7-zip/7z.exe DESTINATION ${CMAKE_BINARY_DIR}/tools/)
file(COPY ${CMAKE_SOURCE_DIR}/config.ini DESTINATION ${CMAKE_BINARY_DIR})
```

* `add_custom_command(TARGET ... POST_BUILD ...)`: 为一个目标（这里是 `${PROJECT_NAME}`）添加一个在构建之后（`POST_BUILD`）执行的自定义命令。 这对于打包、部署或者移动生成的文件非常有用。
    * `COMMAND ${CMAKE_COMMAND} -E ...`: `${CMAKE_COMMAND}` 是 `cmake` 程序本身的可执行文件路径。 `-E` 参数让 CMake 可以执行一些跨平台的命令，例如 `make_directory` (创建目录) 和 `copy_if_different` (如果文件有变化则复制)。
    * `$<TARGET_FILE_DIR:${PROJECT_NAME}>`: 是一个生成器表达式，代表目标文件所在的目录。
    * `COMMENT "..."`: 在执行命令时会在终端打印出的注释信息。

* `make_directory(...)`: 这个命令在 CMake **配置**阶段（即运行 `cmake .` 时）就会执行，直接创建一个目录。
* `file(COPY ...)`: 同样，这个命令也在 CMake **配置**阶段执行，用于复制文件或目录。
    * `${CMAKE_BINARY_DIR}`: 构建目录的路径。
    * `${CMAKE_SOURCE_DIR}`: 源码根目录的路径。

## 使用 `install` 安装目标和文件

### 安装目标（Targets）

这是 `install` 最常见的用法，用于安装由 `add_executable()` 或 `add_library()` 创建的目标。

```cmake
# 创建一个可执行文件和一个共享库
add_executable(my_app main.cpp)
add_library(my_lib SHARED my_lib.cpp)

# 安装可执行文件、库文件和头文件
install(TARGETS my_app my_lib
    RUNTIME DESTINATION bin         # 可执行文件 (.exe, .app) 和 DLL (在Windows上)
    LIBRARY DESTINATION lib         # 共享库 (.so, .dylib) 和静态库 (.a)
    ARCHIVE DESTINATION lib/static  # 静态库 (.a, .lib)
    PUBLIC_HEADER DESTINATION include # C/C++ 头文件
)
```

* **`TARGETS`**: 关键字，后面跟上一个或多个你创建的目标名称。
* **`RUNTIME DESTINATION`**: 指定可执行文件和 Windows 上的 DLL 的安装目录。
* **`LIBRARY DESTINATION`**: 指定共享库 (`.so`, `.dylib`) 的安装目录。
* **`ARCHIVE DESTINATION`**: 指定静态库 (`.a`, `.lib`) 的安装目录。
* **`PUBLIC_HEADER DESTINATION`**: 如果你在 `target_sources` 中标记了 `PUBLIC` 或 `INTERFACE` 的头文件，这个命令会将它们安装到指定位置。你需要先设置 `PUBLIC_HEADER` 属性：

    ```cmake
    set_target_properties(my_lib PROPERTIES
        PUBLIC_HEADER "include/my_lib.h"
    )
    ```

### 使用 `GNUInstallDirs` 定义标准安装路径

硬编码 `bin`, `lib`, `include` 等路径不够灵活。`GNUInstallDirs` 模块提供了符合 GNU 标准的目录变量，能更好地适应不同平台（例如，某些 Linux 发行版会使用 `lib64` 而不是 `lib`）。

```cmake
include(GNUInstallDirs)

install(TARGETS my_app
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(TARGETS my_lib
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES "include/my_lib.h"
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
```

* `include(GNUInstallDirs)`: 引入该模块后，你就可以使用以下标准变量：
    * `CMAKE_INSTALL_BINDIR`: `bin` (可执行文件)
    * `CMAKE_INSTALL_LIBDIR`: `lib` 或 `lib64` 等 (库文件)
    * `CMAKE_INSTALL_INCLUDEDIR`: `include` (头文件)
    * `CMAKE_INSTALL_SYSCONFDIR`: `etc` (配置文件)
    * 等等...
* 这使得你的安装规则更具可移植性和规范性。

### 安装任意文件和目录

除了目标，你经常需要安装配置文件、文档、脚本等。

```cmake
# 安装单个配置文件，并可以重命名
install(FILES config.prod.ini
    DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}
    RENAME config.ini  # 在安装时重命名
)

# 安装整个文档目录
install(DIRECTORY doc/
    DESTINATION share/doc/${PROJECT_NAME}
)

# 安装目录，但排除版本控制文件 (.git, .svn)
install(DIRECTORY assets/
    DESTINATION share/${PROJECT_NAME}/assets
    FILES_MATCHING PATTERN "*"
    PATTERN ".git" EXCLUDE
    PATTERN ".svn" EXCLUDE
)
```

* **`FILES`**: 用于安装一个或多个单独的文件。
* **`DIRECTORY`**: 用于安装整个目录的内容。注意 `doc/` 末尾的斜杠，它表示复制 `doc` 目录的 *内容*，而不是 `doc` 目录本身。
* **`RENAME`**: 在安装时给文件一个新的名字。
* **`FILES_MATCHING`**: 表示只有匹配后续 `PATTERN` 的文件才会被安装。
* **`PATTERN`**: 使用正则表达式来匹配或排除文件/目录。

### 为不同的构建类型（Debug/Release）安装不同的文件

有时你可能只想在 Debug 模式下安装符号文件（`.pdb`），或者为不同的构建类型安装不同的配置文件。

```cmake
# 安装配置文件，根据构建类型选择不同的源文件
install(FILES
    $<BUILD_TYPE:Debug>:${CMAKE_SOURCE_DIR}/config.dev.ini
    $<BUILD_TYPE:Release>:${CMAKE_SOURCE_DIR}/config.prod.ini
    DESTINATION etc
    RENAME config.ini
)

# 只在 Debug 模式下安装 PDB 文件 (MSVC 编译器)
install(FILES
    $<TARGET_PDB_FILE:my_app>
    DESTINATION bin
    CONFIGURATIONS Debug
    OPTIONAL  # 如果文件不存在也不报错
)
```

* **`$<BUILD_TYPE:...>`**: 这是一个生成器表达式，它的值会根据当前的构建类型（`Debug`, `Release`, `RelWithDebInfo`等）而变化。
* **`CONFIGURATIONS`**: 明确指定这条 `install` 规则只对某些构建类型生效。
* **`$<TARGET_PDB_FILE:...>`**: 获取目标 PDB 文件的路径。
* **`OPTIONAL`**: 如果指定的文件不存在，CMake 不会报错。这对于 PDB 文件这类可能不存在的情况很有用。

### 创建导出（Export）配置

这是最强大的技巧之一。如果你希望你的项目（一个库）能被其他 CMake 项目通过 `find_package()` 使用，你就需要创建并安装一个 "导出配置"。

**`CMakeLists.txt`:**

```cmake
# ... 定义 my_lib 目标 ...

install(TARGETS my_lib
    EXPORT MyLibTargets  # 将 my_lib 添加到一个名为 MyLibTargets 的导出集合中
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# 安装导出集合的 .cmake 文件
install(EXPORT MyLibTargets
    FILE MyLibTargets.cmake
    NAMESPACE MyCompany::  # 为导入的目标添加命名空间 (例如 MyCompany::my_lib)
    DESTINATION lib/cmake/MyLib
)

# (可选但推荐) 生成并安装版本配置文件
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "MyLibConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/MyLibConfigVersion.cmake"
    "cmake/MyLibConfig.cmake"  # 这是一个你手动编写的配置文件模板
    DESTINATION lib/cmake/MyLib
)
```

**`cmake/MyLibConfig.cmake` (需要自己创建):**

```cmake
# 包含由 install(EXPORT) 生成的目标文件
include("${CMAKE_CURRENT_LIST_DIR}/MyLibTargets.cmake")
```

* **`EXPORT MyLibTargets`**: 在 `install(TARGETS)` 中使用此选项，会将安装的目标信息收集到一个名为 `MyLibTargets` 的 "导出集合" 中。
* **`install(EXPORT ...)`**: 这个命令会处理 `MyLibTargets` 集合，并生成一个 `MyLibTargets.cmake` 文件。这个文件包含了目标的位置、属性和依赖关系。
* **`NAMESPACE`**: 为导入的目标提供一个命名空间，避免与其他项目的目标发生名称冲突。其他项目使用时就需要写 `MyCompany::my_lib`。
* **版本文件**: `write_basic_package_version_file` 可以自动生成版本检查文件，让 `find_package(MyLib 1.2.0 REQUIRED)` 这样的命令可以正常工作。
* 当其他项目调用 `find_package(MyLib)` 时，CMake 会找到你安装的 `MyLibConfig.cmake` 或 `MyLib-config.cmake`，然后通过 `include` 加载 `MyLibTargets.cmake`，从而神奇地找到你的库、头文件和所有相关设置。
