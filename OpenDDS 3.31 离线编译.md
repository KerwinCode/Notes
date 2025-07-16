# OpenDDS 3.31 离线编译

## 源码准备

1. OpenDDS-3.31.0.tar.gz
2. rapidjson-src（cmake 会自动拉取，位置在`build/_deps/rapidjson-src`，可拷贝到根目录）
3. ACE+TAO-src-7.1.3.tar.bz2
4. ACE+TAO-src-7.1.3.zip（cmake 会自动拉取，这两个文件位置在`build/_deps/ace_tao_dl-tmp`，可拷贝到其他目录，或者更推荐配置到本地 http 服务器上）

## 离线编译步骤

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
        )
        FetchContent_Populate(rapidjson)
        set(OPENDDS_RAPIDJSON "${rapidjson_SOURCE_DIR}")
      endif()
    endif()
    ```

3. `ACE+TAO`依赖处理

    修改源码根目录下的`acetao.ini`，30 行 和 36 行，改为本地路径或本地服务器地址

4. `ACE+TAO`编译时添加`-fPIC`

    `cmake/build_ace_tao.pl` 90 行左右添加：

    ```cmake
    if(UNIX)
      list(APPEND __build_cmd
        "opendds_cmake_pic=-fPIC"
      )
    endif()
    ```

    `cmake/configure_ace_tao.pl` 112 行左右添加：

    ```perl
    print $platform_macros_file ("CCFLAGS += \$(opendds_cmake_pic)\n");
    ```

做完以上步骤，正常编译即可。
