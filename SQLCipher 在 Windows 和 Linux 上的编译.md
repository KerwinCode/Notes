# SQLCipher 在 Windows 和 Linux 上的编译

## SQLCipher 在 Windows 上的编译

### 准备工作

Visual Studio 2022

[SQLCipher Source Code](https://github.com/sqlcipher/sqlcipher/releases)

[OpenSSL Win64 完整包](https://slproweb.com/products/Win32OpenSSL.html)

[IronTCL](https://www.irontcl.com/index.html)

**为避免编译时微软对 UTF-8 等 Unicode 的解读影响 nmake，务必要先将控制面板中更改系统区域设置设定成”English (United States)”，或者勾选使用 Unicode UTF-8 提供全球语言支持（推荐）。**编译与测试完成后再恢复成原来的设定即可。

### 编译过程

先找到 Build Tools for Visual Studio 的目录，如果安装时没有特别修改，一般会在 C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build

开启 Command Prompt，将 directory 移至该目录，依序输入命令

```sh
.\vcvarsall.bat x64

SET PATH=%PATH%;C:\IronTcl\bin

SET PLATFORM=x64
```

将 directory 移至 sqlcipher 的代码目录后创建”BUILD”目录，再 change directory 到里面

```sh
mkdir BUILD

cd BUILD
```

编辑 SQLCipher 的 Makefile 设定，右键使用 Notepad 或 Notepad++打开。

找到 TCC & RCC 的 -DSQLITE_TEMP_STORE 设定，并改成 2 (大概在第 1049 行)，然后再这行后增加以下设定，前后内容如下所示：

```makefile
# Flags controlling use of the in memory btree implementation
#
# SQLITE_TEMP_STORE is 0 to force temporary tables to be in a file, 1 to
# default to file, 2 to default to memory, and 3 to force temporary
# tables to always be in memory.
#
TCC = $(TCC) -DSQLITE_TEMP_STORE=2
RCC = $(RCC) -DSQLITE_TEMP_STORE=2

# Flags to include OpenSSL
TCC = $(TCC) -DSQLITE_HAS_CODEC -I"C:\Program Files\OpenSSL-Win64\include"
```

如果你的 OpenSSL 安装路径不一样，请修改后半部分”C:\Program Files\OpenSSL-Win64\include”至你 OpenSSL 的安装路径。(例 C:\OpenSSL-Win64\include)

接着找到”LTLIBPATHS = $(LTLIBPATHS) /LIBPATH:$(ICULIBDIR)”(大概在第 1265 行)并在其设定结束后的一行（!ENDIF）后加入以下设定，前后内容如下所示：

```makefile
# If ICU support is enabled, add the linker options for it.
#
!IF $(USE_ICU)!=0
LTLIBPATHS = $(LTLIBPATHS) /LIBPATH:$(ICULIBDIR)
LTLIBS = $(LTLIBS) $(LIBICU)
!ENDIF

LTLIBPATHS = $(LTLIBPATHS) /LIBPATH:$(ICULIBDIR) /LIBPATH:"C:\Program Files\OpenSSL-Win64\lib\VC\x64\MT"
LTLIBS = $(LTLIBS) libcrypto.lib libssl.lib ws2_32.lib shell32.lib advapi32.lib gdi32.lib user32.lib crypt32.lib kernel32.lib
# <</mark>>
```

如果你的 OpenSSL 安装路径不一样，也需要修改”C:\Program Files\OpenSSL-Win64\lib\VC\static”，修改逻辑与上一步相同。

修改完后保存，离开，回到 Command Prompt，输入以下指令。

```sh
nmake /f C:\sqlcipher-4.6.1\Makefile.msc TOP=C:\sqlcipher-4.6.1
```

编译完成画面如下图。你也会看到你的 BUILD 文件夹里多了很多文件。

检查 BUILD 文件夹里是否有以下文件

1. sqlite3.exe
2. sqlite3.lib
3. sqlite3.dll
4. sqlite3.h
5. sqlite3.c
6. sqlite3ext.h
7. shell.c

### 使用方法

#### 命令行模式

在 BUILD 文件夹里执行 sqlcipher CLI，输入以下指令即可。

```sh
.\sqlite3.exe
```

看到有“SQLCipher <版本号> community”字眼，如上图红框处才代表有成功运行 SQLCipher。成功运行后，用 .exit 指令退出 SQLCipher CLI

### 参考资料

<https://www.thevoidlabs.com/learn_cpp/cpp_adv_sqlcipher/>

## SQLCipher 在 Linux 上的编译

### 编译环境

WSL2(Ubuntu 22.04.4 LTS)

### 准备工作

[SQLCipher Source Code](https://github.com/sqlcipher/sqlcipher/releases)

```sh
sudo apt-get install build-essential openssl libssl-dev tcl
```

### 编译过程

```sh
./configure --enable-tempstore=yes CFLAGS="-fPIC -DSQLITE_HAS_CODEC -DSQLITE_ENABLE_COLUMN_METADATA" LDFLAGS="/usr/lib/x86_64-linux-gnu/libcrypto.a"

make

make install
```

注意：通常在 linux 下用 gcc 编译动态库时都会加上一个`-fPIC`选项来生成位置无关代码。**在编译静态库时，如果静态库可能会被动态库使用，那么静态库编译的时候就也需要`-fPIC`选项**

## 在 SQLiteCpp 中集成 SQLCipher

### 准备工作

[SQLiteCpp Source Code](https://github.com/SRombauts/SQLiteCpp/releases/)

按上述方法编译 SQLCipher

### 集成过程

将在 Windows 上和 Linux 上编译生成的库文件拷贝到 SQLiteCpp 源码目录下的 sqlcipher 目录中

```sh
sqlcipher
├── libcrypto-3-x64.dll
├── libsqlcipher.a
├── libsqlcipher.so -> libsqlcipher.so.0.8.6
├── libsqlcipher.so.0 -> libsqlcipher.so.0.8.6
├── libsqlcipher.so.0.8.6
├── sqlite3.dll
├── sqlite3.h
└── sqlite3.lib
```

修改 SQLiteCpp/CMakeLists.txt：

```cmake
target_include_directories(SQLiteCpp PRIVATE sqlcipher)
if (WIN32)
 target_link_libraries(SQLiteCpp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/sqlcipher/sqlite3.lib)
else()
    # 链接动态库
 # target_link_libraries(SQLiteCpp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/sqlcipher/libsqlcipher.so)
    # 链接静态库
 target_link_libraries(SQLiteCpp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/sqlcipher/libsqlcipher.a)
endif()

## Build provided copy of SQLite3 C library ##
# 注释掉这一部分
```

修改使用 SQLiteCpp 库的项目的 CMakeLists.txt：

```cmake
target_link_libraries(project_name PRIVATE SQLiteCpp)
if (WIN32)
 target_link_libraries(project_name PRIVATE ${CMAKE_SOURCE_DIR}/thirdparty/SQLiteCpp/sqlcipher/sqlite3.lib)
else()
    # 链接动态库
 # target_link_libraries(project_name PRIVATE ${CMAKE_SOURCE_DIR}/thirdparty/SQLiteCpp/sqlcipher/libsqlcipher.so)
 # 链接静态库
 set(OPENSSL_USE_STATIC_LIBS TRUE)
 find_package(OpenSSL REQUIRED)
 target_link_libraries(project_name PRIVATE
  ${CMAKE_SOURCE_DIR}/thirdparty/SQLiteCpp/sqlcipher/libsqlcipher.a
  OpenSSL::SSL
  OpenSSL::Crypto
 )  # 必须按顺序链接，顺序需遵循“被依赖的库在右侧”原则
endif()
```

## SQLCipher 的使用

### 从 sql 文件创建加密数据库

**以下所有的`sqlcipher` 命令可用集成 SQLCipher 的`sqlite3`替代**

```sh
sqlcipher encrypted.db
PRAGMA key = 'xxxxxx';  # 替换为你的密钥
.read your_script.sql  # 导入 SQL 文件
```

PRAGMA key 是 SQLCipher 中用于设置数据库加密密钥的关键指令，但它**不能直接写入普通的 SQL 文件**中执行。

SQLCipher 要求 `PRAGMA key` 必须在连接数据库后**立即执行**，且在所有其他操作之前。若将此命令写入 SQL 文件并通过常规方式执行（如 `.read` 或批量导入），可能因执行顺序问题导致密钥未正确设置，从而触发错误 `file is encrypted or is not a database`。

### 从 db 文件创建加密数据库

#### **方法一：直接附加数据库（推荐）**

1. **创建加密数据库并附加原始数据库**

    在终端中执行以下命令：

    ```sh
    sqlcipher encrypted.db  # 打开新加密数据库
    PRAGMA key = 'your_encryption_key';  # 设置加密密钥
    PRAGMA cipher_page_size = 4096;      # 可选，需与需求一致
    ATTACH DATABASE 'plaintext.db' AS plaintext KEY '';  # 附加未加密数据库
    SELECT sqlcipher_export('plaintext');  # 将数据导出到加密数据库
    DETACH DATABASE plaintext;           # 分离原始数据库
    ```

    **关键参数解释**：

    - `PRAGMA key`: 加密密钥（支持字符串或十六进制格式，如 `x'2B7E1516'`）。
    - `ATTACH DATABASE`: 附加未加密数据库（`KEY ''` 表示无加密）。
    - `sqlcipher_export('plaintext')`: 将附加库 `plaintext` 的数据复制到当前加密数据库。

2. **验证加密数据库**

    重新打开加密数据库，检查数据完整性：

    ```sh
    sqlcipher encrypted.db
    PRAGMA key = 'your_encryption_key';
    .tables  # 查看表是否存在
    SELECT * FROM your_table LIMIT 1;  # 抽样验证数据
    ```

#### **方法二：通过 SQL 文件中转**

1. **导出未加密数据库的 SQL 结构及数据**

    使用 `sqlite3` 导出原始数据库：

    ```sh
    sqlite3 plaintext.db .schema > backup.sql  # 导出结构
    sqlite3 plaintext.db .dump >> backup.sql   # 导出数据
    ```

2. **创建加密数据库并导入 SQL 文件**

    通过 SQLCipher 导入：

    ```sh
    sqlcipher encrypted.db
    PRAGMA key = 'your_encryption_key';
    .read backup.sql  # 导入 SQL 文件
    ```
